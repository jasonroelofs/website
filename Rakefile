desc "Update rainbow.js files"
task :update_rainbow do
  rainbow_dir = File.join("vendor", "rainbow")
  assets_dir = File.join("assets")

  cd rainbow_dir do
    system "git pull"
  end

  cp File.join(rainbow_dir, "themes", "tricolore.css"), File.join(assets_dir, "css", "syntax")
  cp File.join(rainbow_dir, "js", "rainbow.min.js"), File.join(assets_dir, "js")
  cp File.join(rainbow_dir, "js", "language", "ruby.js"), File.join(assets_dir, "js")
  cp File.join(rainbow_dir, "js", "language", "shell.js"), File.join(assets_dir, "js")
  cp File.join(rainbow_dir, "js", "language", "generic.js"), File.join(assets_dir, "js")
  cp File.join(rainbow_dir, "js", "language", "c.js"), File.join(assets_dir, "js")
end

desc "One-off generate the site"
task :generate do
  sh "rm -rf _site"
  sh "bundle exec jekyll build"
end

desc "Deploy the website"
task :deploy => :generate do
  sh "rsync -rtzq _site/ jameskil@jasonroelofs.com:~/public_html/"
end
