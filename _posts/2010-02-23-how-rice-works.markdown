---
layout: post
title: How Rice Works
tags:
  - everything!
  - programming
  - rice
  - ruby
published: true
---
[Rice](http://rice.rubyforge.org) is one impressive piece of software. I highly doubt I could have written this from scratch (and for that I give Paul Brannan huge accolades for his work) but after working with and on this library for at least a year I probably could. Now that lead maintainership has been passed on to me, I'd like to take some time to explain exactly how Rice works, how one can dive in and help develop this library, and what my plans are for the near future as I work to make Rice and the [rb++ suite](http://rbplusplus.rubyforge.org); a full and proper replacement for [SWIG](http://www.swig.org) when wrapping C++ libraries into Ruby (if you're wrapping a pure C library, I highly recommend [Ruby-FFI](http://github.com/ffi/ffi), though Rice will work).

So with that, I'm putting together this post to help clarify exactly how the various parts of Rice work together, how the entire process flows and more importantly helping anyone who's wanting to learn and work on Rice with a solid starting point (or at least officially crooked). After reading this, anyone should be able to understand the various classes in Rice, and have an idea on where certain functionality lives when looking to add features, or address any bugs. So with that, we'll start at the top.

<h3>What is Rice?</h3>

Rice is a wholly remarkable piece of software that takes complicated C++ classes, methods, types, etc and exposes them into Ruby using as simple an API as can be made. In short, you can take a C++ class such as:

<pre><code data-language="generic">
class Generator {
  public:
    Generator(int seed) : mSeed(seed) { }
    ~Generator() { }

    int getRandomInt() { return 4; } // chosen by dice roll, guaranteed to be random

    void setSeed(int seed) { mSeed = seed; }

  private:
    int mSeed;
};
</code></pre>

and use in in Ruby seemlessly:

<pre><code data-language="ruby">
g = Generator.new(5)
g.random_int          # => 4
g.seed = 10
</code></pre>

through a very simple Rice wrapper Ruby extension:

<pre><code data-language="generic">
void Init_generator() {
   define_class&lt;Generator&gt;("Generator").
      define_constructor(Constructor&lt;Generator, int&gt;()).
      define_method("random_int", &Generator::getRandomInt).
      define_method("seed=", &Generator::setSeed);
}
</code></pre>

Were this class to be wrapped using Ruby's C-API, every method on Generator, and even the constructor would have to be wrapped in a C method with calls into Ruby's API (particularly Data_Make_Struct and Data_Get_Struct), very tedious and time consuming work, and at times quite error prone. So how does Rice make this tedious process as simple as four lines of code? C++ templates, and lots of them. We'll go through each line here and dive into Rice itself to see what's happening, but first, a quick word on Rice's file naming convention.

<h4>ipp, hpp, cpp, _defn ... what??</h4>

Because Rice is so template driven, most of the code cannot be pre compiled. That which can gets put into cpp files and ends up in librice.a, otherwise the core of Rice is template code generation which means only the code needed for the given extension is generated and compiled when building the extension itself. Given this, Rice still tries to strongly adhere to the separation of definition and implementation code. The definitions of classes are normally placed in [class_name]_defn.hpp, with the implementation placed in [class_name].ipp (to keep it separate from compilable code which gets placed in .cpp files). These two files are then combined in [class_name].hpp which is meant to be used in Rice extensions when including various pieces of functionality. When working inside of Rice and you need class definitions, you'll want to use #include "[class_name]_defn.hpp" in most cases or risk running into circular dependencies and "already defined" compiler errors. Otherwise, when writing an extension using Rice, you'll #include "[class_name].hpp" to ensure that all functionality is available for the extension.

As for the .mustache files, a lot of Rice's template code is very repetitive and so instead of using complicated macros like Boost.Python does, Rice simply uses Ruby to generate the C++ code. Rice currently uses the [Mustache](http://defunkt.github.com/mustache/) library for this and the code for this process can be seen in rice/code_gen.

<h4>Defining a class: define_class</h4>

To expose a C++ class into Ruby you first need to define the class, give it a name, and tell Rice which C++ type this class is (as a side note, you can create a Ruby class not hooked to a C++ type, just don't give define_class a type). The prototypes of define_class are found in Data_Type_defn.hpp. The one we are particularly interested in is:

<pre><code data-language="generic">
//! Define a new data class in the default namespace.
/*! The class will have a base class of Object.
 *  \param T the C++ type of the wrapped class.
 *  \return the new class.
 */
template&lt;typename T&gt;
Rice::Data_Type&lt;T&gt; define_class(
    char const * name);
</code></pre>

Following the code into Data_Type.ipp, we can see the implementation of this method:

<pre><code data-language="generic">
template&lt;typename T&gt;
inline Rice::Data_Type&lt;T&gt; Rice::
define_class(
    char const * name)
{
  Class c(define_class(name, rb_cObject));
  c.undef_creation_funcs();
  return Data_Type&lt;T&gt;::template bind&lt;void&gt;(c);
}
</code></pre>

This method starts out creating a new Rice::Class with the given name, making it a subclass of Object (aka a top-level Class, emulating what Ruby itself does). A quick jump into Class.cpp shows us that define_class(char*, VALUE) simply forwards off to the Ruby API call rb_define_class, then takes the resulting VALUE and wraps it into a Rice::Class. 

Rice then undefines #alloc and #initialize to prepare the class for Rice's own type management (to be discussed later). The third line of this method is what actually binds this new class to the C++ type, binding it as a top-level class (Data_Type::bind is templated to the superclass type, so void in this case).

<h4>Data_Type binding</h4>

Probably the most confusing, and at the same time most important, aspect of Rice is how it manages C++ types and their link into the Ruby world. Every C++ class that's to be wrapped into Ruby must be bound to a Ruby class which is done in the static method Data_Type::bind (implemented in Data_Type.ipp). This logic forms the core of how Rice does automatic conversions from C++ types to Ruby types and vis-versa. 

To start, every Data_Type&lt;T&gt; has a static pointer to a Ruby class (Data_Type&lt;T&gt;::klass_). After checking that this C++ class isn't already bound to another Ruby class, bind() registers the Ruby class with the Ruby GC and sets up a Caster between this C++ type and the Ruby class (to be discussed later). 

We've now got our C++ class Generator properly bound to the Ruby class Generator and can continue.

<h4>Adding methods to a class: define_method</h4>

We're now at the point where we can start wrapping the methods on our Generator class. We'll follow the code execution of wrapping the method Generator::getRandomInt(), starting with Module::define_method found in Module_impl.hpp.

**Quick side note:** Rice's C++ API attempts to mimic Ruby's top level Object structure, so the interplay between Data_Type, Module and Class can get confusing. Suffice it to say, class-based objects in Rice are all Rice::Module-s at the top, which then itself is a Rice::Object.

The specific define_method we're interested in here is as follows:

<pre><code data-language="generic">
  //! Define an instance method.
  /*! The method's implementation can be any function or member
   *  function.  A wrapper will be generated which will use from_ruby<>
   *  to convert the arguments from ruby types to C++ types before
   *  calling the function.  The return value will be converted back to
   *  ruby by using to_ruby().
   *  \param name the name of the method
   *  \param func the implementation of the function, either a function
   *  pointer or a member function pointer.
   *  \param arguments the list of arguments of this function, used for
   *  defining default parameters (optional)
   *  \return *this
   */
  template&lt;typename Func_T&gt;
  Derived_T & define_method(
      Identifier name,
      Func_T func,
      Arguments* arguments = 0);
</code></pre>

The Rice::Arguments object has to do with the implementation of default argument definitions and I'll get to that later.

This method is the beginning of where the true template craziness (genius? probably) happens. As a warning, if you are not comfortable with templates, the following is either going to confuse you, or it will open your eyes to a whole new level of possibilities in C++.

Rice works directly with Function Pointers and template definitions tailored to react to specific function pointer definitions to generate the appropriate code for properly wrapping C++ methods into Ruby. As a refresher, a C++ function pointer looks like this:

<pre><code data-language="generic">
ReturnType (ClassName::*methodName)(ArgT1, ArgT2, ...);
</code></pre>

so if we were to print out the function pointer of Generator::getRandomInt we would see:

<pre><code data-language="generic">
int (Generator::*getRandomInt)();
</code></pre>

This is what Func_T ends up being, a function pointer type. It can be a method with any return type, and any number and type of arguments, all handled by this one method. Looking at the implementation of this in Module_impl.ipp, we can see that execution is simply forwarded off to detail::define_method_and_auto_wrap given the Module/Class/Data_Type (*this), the name, the function pointer, any defined exception handler (this-&gt;handler) and the arguments object.

From this point, Rice needs to make a very important distinction on the method being wrapped: is it a class method, or a static method? The method Rice::detail::define_method_and_auto_wrap, defined in detail/define_method_and_auto_wrap.{hpp,ipp} uses Wrapped_Function classes, via wrap_function (detail/wrap_function.{hpp,ipp}) to make this distinction and to set up the final method object before finally jumping into Ruby's C-API for finally exposing this method.

<h4>Auto_{,Member}_Function_Wrapper</h4>

Opening up detail/wrap_function.ipp you'll see a very long file full of very repetitive method definitions. The key here is this method uses templates to match the passed in function pointer to one of Auto_Function_Wrapper (static functions) or Auto_Member_Function_Wrapper (a class instance method). Due to how Auto_{,Member}_Function_Wrapper works, wrap_function also has to have template specifications for functions with a return type and functions with void, as well as specifications to handle const and non-const function pointers.

To continue from here we need to understand how Rice actually hooks into Ruby. Glancing in detail/Auto_Function_Wrapper.hpp  we'll find template-overload classes similar to what's in wrap_function. These classes are the core that handle argument and return type management as well as taking execution from Ruby and passing it into the related C++ function or method. What we're going to focus on here is the ::call static method, though we'll get to the details of this method's implementations later. 

Ruby's API will only take C-style functions with the signature VALUE(*)(...). The Auto_..._Wrapper classes are the glue that gives us a function with this signature:

<pre><code data-language="generic">
static VALUE Auto_Function_Wrapper::call(int argc, VALUE *argv, VALUE self);
</code></pre>

If you're familiar with Ruby's C-API, then this signature should look familiar, it's the -1 arity signature used when you want to support a variable number of arguments (in Rice's case, default arguments). So how does a static function on a templated class work with an instance of this class to call the correct C++ method or function? It's time to dive into detail/method_data.cpp, which defines define_method_with_data() as seen in the second part of define_method_and_auto_wrap() (Rice::protect is a wrapper around rb_protect to catch any exceptions and handle them gracefully).

<h4>detail/method_data.cpp</h4>

I'll start this section with the comment block describing define_method_with_data():

<pre><code data-language="generic">
// Define a method and attach data to it.
// The method looks to ruby like a normal aliased CFUNC, with a modified
// origin class.
//
// How this works:
//
// To store method data and have it registered with the GC, we need a
// "slot" to put it in.  This "slot" must be recognized and marked by
// the garbage collector.  There happens to be such a place we can put
// data, and it has to do with aliased methods.  When Ruby creates an
// alias for a method, it stores a reference to the original class in
// the method entry.  The form of the method entry differs from ruby
// version to ruby version, but the concept is the same across all of
// them.
// 
// In Rice, we make use of this by defining a method on a dummy class,
// then attaching that method to our real class.  The method is a real
// method in our class, but its origin class is our dummy class.
// 
// When Ruby makes a method call, it stores the origin class in the
// current stack frame.  When Ruby calls into Rice, we grab the origin
// class from the stack frame, then pull the data out of the origin
// class.  The data item is then used to determine how to convert
// arguments and return type, how to handle exceptions, etc.
</pre></code>

Bear with me here as I still have a hard time keeping this straight in my mind. To help understand what exactly is going on here, I've made a graph showing all the relevant links between Rice, C++ and Ruby as made by this method.

<img src="http://jameskilton.com/blog/wp-content/uploads/2010/02/method_data.png" alt="Graph showing how Rice hooks into Ruby" />

The steps define_method_with_data takes:

   1. Create a new class inside of the class we're exposing (called the origin class).
   2. Build a new two-element array that has [ 0xDA7A, VALUE pointer to Auto_..._Wrapper class]
   3. Give the origin class an easily recognized name (which is also an invalid Ruby constant name, preventing it from every being accidentally used in Ruby code, but it prints out in object dumps on occasion).
   4. Set this origin class to be FT_SINGLETON which hides it from the user ((<a href="http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-talk/336728">http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-talk/336728</a>)).
   5. Set a hidden ivar on the class that's the array built in step 2
   6. Define a method on the origin class that links to Auto_..._Wrapper::call with the requested name

At this point we've got everything ready except for actually being able to call the method from Ruby. These next steps were hard to show right on the graph because Rice actually modifies the symbol table of the base class in question.

  7. Create an alias on the origin class that aliases the method to "dummy"
  8. Find this new alias entry in the origin class' symbol table
  9. Modify the symbol table of the base class to call "dummy" on the origin class on any call to the method being wrapped.

We've now got everything hooked up: we have the c-style function hooked into Ruby and we have a way to access the actual Auto_..._Wrapper class once call() gets called. Now, you may be asking why this is such a complicated process when it could be done with a simple rb_define_method on Auto_..._Wrapper::call(). The answer to this question is show in the image above, there's data and information saved in this Auto_..._Wrapper object that Rice needs to be able to properly convert argument and return types, and handle any exceptions that may get thrown.

With that, we're done with the wrapping and exposing path through Rice. Now it's time to head back up the chain and see how Rice handles calls from Ruby and passing execution back up into C++.

<h4>Auto_..._Wrapper::call()</h4>

It's time to take a detailed look at this method. In short, once execution enters this method, it's now Rice's job to convert VALUE arguments into the appropriate C++ types, execute the wrapped method / function, then do any required conversion of return values back to VALUEs for Ruby, while also making sure that any exceptions that throw along the way are properly caught and handled (an uncaught exception in C++ will give you a SegmentationFault or a BusError. Always fun to debug).

So, first things first, we need to get back the Auto_..._Wrapper object we saved when we wrapped this function. This is done with the detail::method_data() function, details of which are easily seen and understood in detail/method_data.cpp. With this we run our arguments list through rb_scan_args then run through each argument and check if there's was a default set in the wrapper and if we should use it (Side note: the sanitize&lt;&gt; template is a no-op right now and can be ignored. It's there to attempt to clean out any references or const arguments that may come in to get the base type, but this was causing crashes on other wrappers. For now I assume there's a possibility of a small memory leak here related to reference types). Once all the types are converted it's time to actually call the C++ method or function, and handle the return type. 

The noticeable difference between Auto_Member_Function_Wrapper and Auto_Function_Wrapper is that Auto_Function_Wrapper has some special handling for the "self" parameter. This functionality is here to support the ability to wrap C functions that are written in an OO way, such as:

<pre><code data-language="generic">
Jukebox* newJukebox();
void play(Jukebox* jukebox, Song* song);
void stop(Jukebox* jukebox);
void addSong(Jukebox* jukebox, Song* song);
// etc.
</code></pre>

Because of the special self handling of Auto_Function_Wrapper, these methods can be wrapped into a Jukebox class very simply:

<pre><code data-language="generic">
define_class("Jukebox").
  define_method("initialize", &newJukebox).
  define_method("play", &play).
  define_method("stop", &stop).
  define_method("add_song", &addSong);
</code></pre>

Other than this, the two classes are identical in functionality.

<h4>Constructor</h4>

Because constructors aren't accessible via normal function pointers, Rice has to implement special handling of wrapping class constructors for proper instantiation. This is done through the Rice::Constructor class and the define_constructor method. The path here is mostly equivalent to define_method. Opening up Constructor.{hpp,ipp} you can see the same ::call() static method, with a much simpler body. Every Ruby object has a free slot accessible through DATA_PTR. It's in this slot that C++ class instances are saved and used for type conversions as described next. Constructor::call creates this object and saves it in DATA_PTR for later use.

<h4>Default Arguments</h4>

The code for handling and managing Default Arguments is pretty easy to read, but how the syntax work is definitely cause for confusion. Pulled from Boost.Python's implementation of this exact feature, this functionality uses the comma operator to string together Rice::Arg objects into a single Arguments object that's then passed into the Wrapped_Function object for that method. Due to the fact the comma operator doesn't even come into play if there's a single argument to a method, I was forced to copy some functionality around to make this feature work in all situations. So where you see two methods, one which takes a Rice::Arguments* and one which takes a Rice::Arg*, that's what's going on. I hope to clean this up drastically in the near future.

<h4>to_ruby / from_ruby</h4>

The last piece of Rice's functionality is the type conversion system, exposed to the user through two methods: to_ruby(), which takes a C/C++ type and converts it to a VALUE, and from_ruby() which does the opposite. The signatures of these methods are simple:

<pre><code data-language="generic">
template&lt;typename T&gt;
T from_ruby(Rice::Object x);

template&lt;typename T&gt;
Rice::Object to_ruby(T const & x);
</code></pre>

For most basic and fundamental types, the to_/from_ruby implementations are very simple and can be seen in to_from_ruby.ipp, but this is only half of the story.

Rice's real strength comes in handling custom-defined C++ types and seamlessly converting them from Ruby to C++ and back again, making sure to handle inheritance (both C++ &lt;=&gt; C++ and C++ &lt;=&gt; Ruby) appropriately. The core of this functionality is found in Data_Type::from_ruby in Data_Type.ipp, and in the Rice::Caster class found in detail/Caster.hpp, but it begins at the end of Data_Type::bind, which creates a new Caster for the newly bound type:

<pre><code data-language="generic">
  detail::Abstract_Caster * base_caster = Data_Type&lt;Base_T&gt;().caster();
  caster_.reset(new detail::Caster&lt;T, Base_T&gt;(base_caster, klass));
  Data_Type_Base::casters().insert(std::make_pair(klass, caster_.get()))
</code></pre>

Caster is at the core very simple. It's a class that enables Rice to build a tree of Casters that define how to cast from a Ruby type into a custom C++ type. The code above is simply building a new Caster linking the new type to the Ruby class, specifying any superclass type as needed, then registering this Caster with Rice's Caster tree. From here it's time to look at how Rice uses this Caster list in Data_Type::from_ruby() (which btw is called through ::convert() in detail/from_ruby.{hpp,ipp}).

In the case that the C++ type we want is exactly the Ruby type we've been given, casting is easy, make a new Data_Object for that type and get the pointer. If this doesn't match, then we need to start heading up the inheritance heirarchy (rb_mod_ancestors) looking for the <strong>lowest</strong> class in the hierarchy bound in Rice, get it's Caster and attempt to cast into C++. This functionality allows any depth of inheritance to work seamlessly in Rice.

<h3>The Future of Rice</h3>

Rice is currently a very capable library, and is being used successfully by a couple of projects, but it does have a long way to go to be the SWIG replacement (along with my rb++ suite) I want it to be. For the next release of Rice I'm current planning on the following work:

  * Refactoring Auto_{,Member}_Function_Wrapper, Constructor, wrap_function, et.al. to be much simpler. Going to try to abstract out the actual method dispatch into a class like Auto_Functor_Wrapper which will handle all the to_/from_ruby and argument management magic.
  * Default arguments on Constructor. To do this Constructor needs to act more like an Auto_Function_Wrapper, and thus would be much easier to implement once the refactoring above is done.
  * Beginning into a def_helper (from Boost.Python) feature. Right now the implementation of default arguments is pretty messy. As I need to handle multiple arguments as well as a single argument, I've resorted to two define_method implementations to take these two cases. With some template magic, I can make it work with a single argument, as well as opening up a single place for call and return policy functionality, instead of multiple places that would be required today.

As always, I'm available over Github messages / Issues, Email, comments here, or on Rice's mailing list (rice AT librelist DOT com).
