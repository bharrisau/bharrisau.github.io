require 'rubygems'
require 'rake'
require 'rdoc'
require 'date'
require 'yaml'
require 'tmpdir'
require 'jekyll'

desc "Generate blog files"
task :generate do
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end


desc "Generate and publish blog to gh-pages"
task :publish => [:generate] do
  Dir.mktmpdir do |tmp|
    system "mv _site/* #{tmp}"
    system "git stash"
    # system "git branch -D master"
    system "git checkout master"
    system "rm -rf *"
    system "rm -rf .sass-cache"
    system "mv #{tmp}/* ."
    message = "Site updated at #{Time.now.utc}"
    system "git add ."
    system "git commit -am #{message.shellescape}"
    system "git push origin master"
    system "git checkout source"
    system "git stash pop"
    system "echo Published"
  end
end

desc "compile and run the site"
task :serve do
  pids = [
    spawn("jekyll serve"), # put `auto: true` in your _config.yml
    spawn("compass watch")
    # spawn("coffee -b -w -o javascripts -c assets/*.coffee")
  ]
   
  trap "INT" do
    Process.kill "INT", *pids
    exit 1
  end
   
  loop do
    sleep 1
  end
end

task :default => :serve
