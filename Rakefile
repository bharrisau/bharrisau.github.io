# Parts copied from David Ensinger
# http://davidensinger.com/2013/07/automating-jekyll-deployment-to-github-pages-with-rake/

require 'rubygems'
require 'rake'
require 'rdoc'
require 'date'
require 'yaml'
require 'tmpdir'
require 'jekyll'

desc "Generate blog files"
task :generate do
  puts "\n## Generating production site"
  system "touch -m sass/app.scss"
  system "compass compile -e production"
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end

desc "Commit _site/"
task :commit => [:generate] do
  puts "\n## Staging modified files"
  status = system("git add -A")
  puts status ? "Success" : "Failed"
  puts "\n## Committing a site build at #{Time.now.utc}"
  message = "Build site at #{Time.now.utc}"
  status = system("git commit -m \"#{message}\"")
  puts status ? "Success" : "Failed"
  puts "\n## Pushing commits to remote"
  status = system("git push origin source")
  puts status ? "Success" : "Failed"
end

desc "Generate and publish blog to gh-pages"
task :publish => [:commit] do
  puts "\n## Deleting master branch"
  status = system("git branch -D master")
  puts status ? "Success" : "Failed"
  puts "\n## Creating new master branch and switching to it"
  status = system("git checkout -b master")
  puts status ? "Success" : "Failed"
  puts "\n## Forcing the _site subdirectory to be project root"
  status = system("git filter-branch --subdirectory-filter _site/ -f")
  puts status ? "Success" : "Failed"
  puts "\n## Switching back to source branch"
  status = system("git checkout source")
  puts status ? "Success" : "Failed"
  puts "\n## Pushing all branches to origin"
  status = system("git push --all origin")
  puts status ? "Success" : "Failed"
end

desc "compile and run the site"
task :serve do
  system "touch -m sass/app.scss"
  pids = [
    spawn("jekyll serve --watch --drafts"), # put `auto: true` in your _config.yml
    spawn("compass watch -e development")
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
