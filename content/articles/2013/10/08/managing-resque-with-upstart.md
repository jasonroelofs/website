---
title: Managing Resque with Upstart
date: '2013-10-08T12:00:00-05:00'
draft: false
titleOnly: true
aliases:
  - /2013/10/08/managing-resque-with-upstart
---

This is a long overdue update to my previous post [Managing Resque with Upstart and Monit](/articles/2012/03/12/manage-and-monitor-resque-with-upstart-and-monit). Much has changed in the world of process management and system administration and it's time to update my previous post with what I see is the current Best Practice.

First, Monit is gone. I haven't installed or used this tool in a long time. That isn't to say Monit is a bad tool, I've just come to see it is a redundant layer. Over the years I don't remember ever receiving an alert that was worth acting on; it has all been noise. Along those lines, I've never had problems keeping Resque alive, and if it's my code that causes Resque workers to die, exception handlers and other application monitors have always told me so very quickly.

Besides the alerting capabilities, the main reason I recommended Monit in the previous post was as a way to to manage multiple Resque processes with a single command. I needed to be able to start/stop/restart all Resque processes without knowing how many a server was running and Monit groups worked very well for that. Now, however, I've seen better ways of using Upstart to manage multiple processes, so Monit is no longer required in this setup.

This new way actually comes from the new kid in the background-jobs scene: [Sidekiq](http://sidekiq.org/). Sidekiq is API compatible with Resque, can be run in multiple processes like Resque, but is meant to be run in a threaded mode. Threaded mode leads to more efficient use of resources especially when under JRuby or Rubinius (due to the lack of a GIL). The Sidekiq project includes a multitude of management scripts, covering SysV, Runit, and of course Upstart. You can find all of the scripts in the [examples directory](https://github.com/mperham/sidekiq/tree/master/examples) of the project. For our case, check out the [upstart](https://github.com/mperham/sidekiq/tree/master/examples/upstart) directory, where there are two sets of scripts, one set for managing [multiple apps and workers](https://github.com/mperham/sidekiq/blob/master/examples/upstart/workers.conf) and a set for managing a [single app](https://github.com/mperham/sidekiq/blob/master/examples/upstart/sidekiq.conf). The key to these scripts is Upstart scripts calling other Upstart scripts, something that didn't occur to me back then. Adapting to Resque shouldn't be difficult, and if I do such a thing I'll be sure to update this post with the scripts I used.

So in summary, use Resque or Sidekiq and Upstart. Nothing else required.
