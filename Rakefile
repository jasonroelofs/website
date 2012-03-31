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
end
