require "rubygems"
require "bundler/setup"

deploy_dir = '_site'
deploy_branch = 'master'

desc "Generate jekyll site"
task :generate do
  puts "## Generating Site with Jekyll"
  system "jekyll build"
end

desc "Default deploy task"
task :deploy do
  puts "## Start Deploying branch to Github Pages "
  cd "#{deploy_dir}" do
    system "git add -A"
    puts "\n## Committing: Site updated at #{Time.now.utc}"
    message = "Site updated at #{Time.now.utc}"
    system "git commit -m \"#{message}\""
    puts "\n## Pushing generated #{deploy_dir} website"
    system "git push -f origin #{deploy_branch}"
    puts "\n## Github Pages deploy complete"
  end
end

desc "Generate jekyll site & Deploy"
task :gen_deploy do
  Rake::Task[:generate].execute
  Rake::Task[:deploy].execute
end


desc "Push Source code to github"
task :push_code do
  system "git add -A"
  puts "\n## Committing: Site updated at #{Time.now.utc}"
  message = "Site updated at #{Time.now.utc}"
  system "git commit -m \"#{message}\""
  system "git push -f origin"
  puts "\n## Code push complete"
end

desc "new post with title, eg: rake new_post title=TITLE"
task :new_post do
  system "thor jekyll:new \"#{ENV['title']}\""
end
