---
title: "How to Bootstrap Your Rails App on a New Server"
date: 2014-09-20 22:31:00 +0800
tags: [ruby]
---

如何在一台全新新的主机 Rails 服务呢？ 有什么比较好的策略？ 有哪些问题需要我们注意？  

这篇文章转载于同事（[小光](https://github.com/serco-chen)）的经验总结，特在此分享。如果在你实践过程中，发现有部分内容过时，请告知一下，谢谢。

#### 0. Setup user for deployment
```
# Install openssh-server
sudo apt-get update
sudo apt-get install openssh-server  # This step may not required in Aliyun host
# Setup deployment user
sudo adduser deploy  # Add a user for deployment
sudo usermod -a -G sudo deploy
#copy pub key to server
ssh deploy@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
cd /etc/ssh
sudo vi sshd_config
# modify sshd_config
PermitRootLogin no
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile %h/.ssh/authorized_keys
# restart ssh server
sudo service ssh restart
```

#### 1. Install packages
```
sudo apt-get update
sudo apt-get install -y build-essential openssl curl libcurl3-dev libreadline6 libreadline6-dev git zlib1g zlib1g-dev libssl-dev libyaml-dev libxml2-dev libxslt-dev autoconf automake libtool imagemagick libmagickwand-dev libpcre3-dev libsqlite3-dev libmysql-ruby libmysqlclient-dev
sudo apt-get install git
# 添加ssh_key到github账号上
git config --global user.name "lohaswork"
git config --global user.email "support@lohaswork.com"
ssh-keygen -t rsa -C "support@lohaswork.com"
cat ~/.ssh/id_rsa.pub   //复制pub-key至github账号设定的ssh keys中
ssh -T git@github.com   //测试ssh至github是否成功
# Make deployment directory
sudo mkdir /www
sudo mkdir /www/teamind_deploy
sudo chown -R deploy /www/teamind_deploy
cd home/deploy
git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
git clone git://github.com/sstephenson/rbenv-gem-rehash.git ~/.rbenv/plugins/rbenv-gem-rehash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> .profile
echo 'export PATH="$HOME/.rbenv/shims:$PATH"' >> .profile
echo 'eval "$(rbenv init -)"' >> .profile
source ~/.bash_profile
# Reboot
sudo reboot
# Upload ruby_build binary package to server using sftp and then
mkdir ~/.rbenv/cache
cp ruby-1.9.3-p448.tar.gz ~/.rbenv/cache
cp yaml-0.1.4.tar.gz ~/.rbenv/cache
rbenv install 1.9.3-p448
rbenv global 1.9.3-p448
gem source -r https://rubygems.org/
gem source -a http://ruby.taobao.org/
# 在 ~/.gemrc 中配置不生成 ri 和 rdoc 文档
$ vi ~/.gemrc
# add following 2 lines to .gemrc
install: --no-rdoc --no-ri
update: --no-rdoc --no-ri
gem install bundler
sudo apt-get install python-software-properties
sudo apt-add-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
sudo apt-get install nginx
sudo service nginx start
cd /etc/nginx/sites-enabled/
sudo rm default
```
#### 2. Problems with locale
```
# ls 初始化月份字符串出错
sudo vi /var/lib/locales/supported.d/local
# 改成如下内容
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GBK GBK
zh_CN GB2312
# 执行
sudo locale-gen
sudo vi /etc/default/locale
# 改成如下内容
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN.UTF-8"
LC_ALL="zh_CN.UTF-8"
```
#### 3. Install database
```
# PostgreSQL
sudo apt-get install postgresql-9.1
sudo apt-get install postgresql-client-9.1 postgresql-contrib-9.1 postgresql-server-dev-9.1
# 配置postgres部署时的用户
sudo passwd postgres
sudo su postgres
postgres=# psql
postgres=# ALTER USER postgres PASSWORD 'postgres'; // 修改密码
postgres=# CREATE USER deploy WITH PASSWORD 'lohaswork' CREATEDB; // 创建deploy用户
postgres=# \q // 退出
exit //go back from user postgres to deploy
sudo service postgresql start
# MySQL
sudo apt-get install mysql-server-5.5 mysql-client-5.5
mysql -uroot -p
mysql> CREATE USER 'deploy'@'localhost' IDENTIFIED BY 'lohaswork';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'deploy'@'localhost' WITH GRANT OPTION;
mysql> \q
```
#### 4. Autoload services
```
# unicorn
cd /etc/init.d
sudo touch nicorn.teamind_deploy
sudo vi unicorn.teamind_deploy
# add following lines to unicorn.teamind_deploy

#!/bin/sh
set -e
# Example init script, this can be used with nginx, too,
# since nginx and unicorn accept the same signals

# Feel free to change any of the following variables for your app:
TIMEOUT=${TIMEOUT-60}
APP_ROOT=/www/teamin_deploy/current
PID=$APP_ROOT/tmp/pids/unicorn.pid
CMD="/usr/bin/unicorn -D -c $APP_ROOT/config/unicorn.rb"
action="$1"
set -u

old_pid="$PID.oldbin"

cd $APP_ROOT || exit 1

sig () {
  test -s "$PID" && kill -$1 `cat $PID`
}

oldsig () {
  test -s $old_pid && kill -$1 `cat $old_pid`
}

case $action in
start)
  sig 0 && echo >&2 "Already running" && exit 0
  $CMD
  ;;
stop)
  sig QUIT && exit 0
  echo >&2 "Not running"
  ;;
force-stop)
  sig TERM && exit 0
  echo >&2 "Not running"
  ;;
restart|reload)
  sig HUP && echo reloaded OK && exit 0
  echo >&2 "Couldn't reload, starting '$CMD' instead"
  $CMD
  ;;
upgrade)
  if sig USR2 && sleep 2 && sig 0 && oldsig QUIT
  then
    n=$TIMEOUT
    while test -s $old_pid && test $n -ge 0
    do
      printf '.' && sleep 1 && n=$(( $n - 1 ))
    done
    echo

    if test $n -lt 0 && test -s $old_pid
    then
      echo >&2 "$old_pid still exists after $TIMEOUT seconds"
      exit 1
    fi
    exit 0
  fi
  echo >&2 "Couldn't upgrade, starting '$CMD' instead"
  $CMD
  ;;
reopen-logs)
  sig USR1
  ;;
*)
  echo >&2 "Usage: $0 <start|stop|restart|upgrade|force-stop|reopen-logs>"
  exit 1
  ;;
esac
```

### 5. deploy.rb nginx.conf unicorn.rb capistrano-db-rollback.rb capistrano_database_yml.rb config.ru

```
#### deploy.rb
require 'bundler/capistrano'
$:.unshift('./config')
require 'capistrano-db-rollback'
require 'capistrano_database_yml'

# Five steps to run the first deployment
# 0. Create unicorn related files at local development ENV
# 1. Create deploy_user in linux and install packages
# 2. Manually create deploy_user in postgres and create www/#{appname} directory
# 3. Modify server info in deploy.rb & nginx.conf & capistrano_database_yml.rb
# 4. Run deploy:setup and config database following the leading message
# 5. !Important: Run cap deploy:cold for the very first deployment
# 6. Upload the video manually

# Need change before deployment
set :server_name, "192.168.1.114"
set :user, "deploy"
set :sudo_user, "deploy"
set :deploy_to, "/www/teamind_deploy"

# Repository
set :application, "LohasWork.com"
set :scm, :git
set :repository,  "git@github.com:lohaswork/LohasWork.com"
set :branch, "serco/for-deploy"  # Need changge to master

# Configurations
set :rails_env, "production"
set :deploy_via, :remote_cache
set :use_sudo, false
set :normalize_asset_timestamps, false
default_run_options[:pty] = true
set :rbenv_version, ENV['RBENV_VERSION'] || "1.9.3-p327"
set :default_environment, {
  'PATH' => "/home/#{user}/.rbenv/shims:/home/#{user}/.rbenv/bin:$PATH",
  'RBENV_VERSION' => "#{rbenv_version}",
}

# Roles
role :web, "192.168.1.114"                          # Your HTTP server, Apache/etc
role :app, "192.168.1.114"                          # This may be the same as your `Web` server
role :db,  "192.168.1.114", :primary => true        # This is where Rails migrations will run

# For Unicorn service
set :unicorn_config, "#{current_path}/config/unicorn.rb"
set :unicorn_pid, "#{current_path}/tmp/pids/unicorn.pid"

namespace :deploy do

  task :cold do       # Overriding the default deploy:cold
    update
    setup_db       # My own step, replacing migrations.
    start
  end

  task :add_shared do
    run "mkdir -p #{shared_path}/public"
    run "chmod g+rx,u+rwx #{shared_path}/public"
    run "mkdir -p #{shared_path}/public/videos"
    run "chmod g+rx,u+rwx #{shared_path}/public/videos"
    run "mkdir -p #{shared_path}/"
    run "chmod g+rx,u+rwx #{shared_path}/tmp"
    run "mkdir -p #{shared_path}/unicorn"
    run "chmod g+rx,u+rwx #{shared_path}/unicorn"
    run "cd #{shared_path}"
    run "touch err.log out.log"
    run "mkdir -p #{shared_path}/sockets"
    run "chmod g+rx,u+rwx #{shared_path}/sockets"
    run "mkdir -p #{shared_path}/tmp/sessions"
    run "chmod g+rx,u+rwx #{shared_path}/tmp/sessions"
    run "mkdir -p #{shared_path}/tmp/cache"
    run "chmod g+rx,u+rwx #{shared_path}/tmp/cache"
  end

  task :start, :roles => :app, :except => { :no_release => true } do
    run "cd #{current_path} && RAILS_ENV=production bundle exec unicorn_rails -c #{unicorn_config} -D"
  end

  task :stop, :roles => :app, :except => { :no_release => true } do
    run "if [ -f #{unicorn_pid} ]; then kill -QUIT `cat #{unicorn_pid}`; fi"
  end

  task :restart, :roles => :app, :except => { :no_release => true } do
    # 用USR2信号来实现无缝部署重启
    run "if [ -f #{unicorn_pid} ]; then kill -s USR2 `cat #{unicorn_pid}`; fi"
  end

  desc 'clean old files, link shared files'
  task :housekeeping, :roles => :app do
    run "rm -rf #{current_path}/public/videos"  ###
    run "ln -s #{shared_path}/public/videos #{current_path}/public/videos"
    run "#{sudo} ln -nfs #{current_path}/config/nginx.conf /etc/nginx/sites-enabled/nginx.conf"
    run "rm -rf #{current_path}/unicorn"
    run "ln -s #{shared_path}/unicorn/ #{current_path}/unicorn"
    run "rm -rf #{current_path}/tmp/sockets"
    run "ln -s #{shared_path}/sockets #{current_path}/tmp/sockets"
    run "rm -rf #{current_path}/tmp/sessions"
    run "ln -s #{shared_path}/tmp/sessions #{current_path}/tmp/sessions"
    run "rm -rf #{current_path}/tmp/cache"
    run "ln -s #{shared_path}/tmp/cache #{current_path}/tmp/cache"

  end

  # utilize that capistrano has already done this!
  #
  # If you put your shared file or folder here:
  #   /path/to/app/shared/sockets
  # Then it will be symlinked here:
  #   /path/to/app/releases/20120517191233/tmp/sockets
  #
  shared_children.push "tmp/sockets"
  shared_children.push "unicorn"

  task :nginx_restart, :roles => :app do
    run "#{sudo} service nginx restart"
  end

  task :setup_db, :roles => :app do
    raise RuntimeError.new('db:setup aborted!') unless Capistrano::CLI.ui.ask("About to `rake db:setup`. Are you sure to wipe the entire database (anything other than 'yes' aborts):") == 'yes'
    run "cd #{current_path}; bundle exec rake db:setup RAILS_ENV=#{rails_env}"
  end

end

after 'deploy:setup', 'deploy:add_shared'
after 'deploy:create_symlink', 'deploy:housekeeping'
after 'deploy:restart', 'deploy:cleanup'
```

```
#### nginx.conf

#每台机器都运行nginx+unicorn，本机用domain socket，方便切换
upstream ruby_backend {
    server unix:/tmp/unicorn.sock fail_timeout=0;
}

#用try_files方式和proxy执行rails动态请求
server {
    listen       80;
    server_name  192.168.0.105;
    root         /www/teamind_deploy/current/public;

    try_files $uri/index.html $uri.html $uri @httpapp;

    location @httpapp {
      proxy_redirect     off;
      proxy_set_header   Host $host;
      proxy_set_header   X-Forwarded-Host $host;
      proxy_set_header   X-Forwarded-Server $host;
      proxy_set_header   X-Real-IP        $remote_addr;
      proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      proxy_buffering    on;
      proxy_pass         http://ruby_backend;
   }
}
```


#### unicorn.rb

```
worker_processes 6 #依据需要的内存数量设定

app_root = File.expand_path("../..", __FILE__)
working_directory app_root

# Listen on fs socket for better performance
listen "/tmp/unicorn.sock", :backlog => 64
listen 4096, :tcp_nopush => false

# Nuke workers after 30 seconds instead of 60 seconds (the default)
timeout 30

# App PID
pid "#{app_root}/tmp/pids/unicorn.pid"

# By default, the Unicorn logger will write to stderr.
# Additionally, some applications/frameworks log to stderr or stdout,
# so prevent them from going to /dev/null when daemonized here:
stderr_path "#{app_root}/log/unicorn.stderr.log"
stdout_path "#{app_root}/log/unicorn.stdout.log"

# To save some memory and improve performance
preload_app true
GC.respond_to?(:copy_on_write_friendly=) and
  GC.copy_on_write_friendly = true

# Force the bundler gemfile environment variable to
# reference the Сapistrano "current" symlink
before_exec do |_|
  ENV["BUNDLE_GEMFILE"] = File.join(app_root, 'Gemfile')
end

before_fork do |server, worker|
  # 参考 http://unicorn.bogomips.org/SIGNALS.html
  # 使用USR2信号，以及在进程完成后用QUIT信号来实现无缝重启
  old_pid = app_root + '/tmp/pids/unicorn.pid.oldbin'
  if File.exists?(old_pid) && server.pid != old_pid
    begin
      Process.kill("QUIT", File.read(old_pid).to_i)
    rescue Errno::ENOENT, Errno::ESRCH
      # someone else did our job for us
    end
  end

  # the following is highly recomended for Rails + "preload_app true"
  # as there's no need for the master process to hold a connection
  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!
end

after_fork do |server, worker|
  # 禁止GC，配合后续的OOB，来减少请求的执行时间
  GC.disable
  # the following is *required* for Rails + "preload_app true",
  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.establish_connection
end
```

#### capistrano-db-rollback.rb
```
configuration = Capistrano::Configuration.respond_to?(:instance) ?
  Capistrano::Configuration.instance(:must_exist) :
  Capistrano.configuration(:must_exist)

configuration.load do
  namespace :deploy do
    namespace :rollback do
      desc <<-DESC
      Rolls back the migration to the version found in schema.rb file of the previous release path.\
      Uses sed command to read the version from schema.rb file.
      DESC
      task :migrations do
        run "cd #{current_release};  rake db:migrate RAILS_ENV=#{rails_env} VERSION=`grep \":version =>\" #{previous_release}/db/schema.rb | sed -e 's/[a-z A-Z = \> \: \. \( \)]//g'`"
      end
      after "deploy:rollback","deploy:rollback:migrations"
    end
  end
end
```
#### capistrano_database_yml.rb
```
#
# = Capistrano database.yml task
#
# Provides a couple of tasks for creating the database.yml
# configuration file dynamically when deploy:setup is run.
#
# Category::    Capistrano
# Package::     Database
# Author::      Simone Carletti <weppos@weppos.net>
# Copyright::   2007-2010 The Authors
# License::     MIT License
# Link::        http://www.simonecarletti.com/
# Source::      http://gist.github.com/2769
#
#
# == Requirements
#
# This extension requires the original <tt>config/database.yml</tt> to be excluded
# from source code checkout. You can easily accomplish this by renaming
# the file (for example to database.example.yml) and appending <tt>database.yml</tt>
# value to your SCM ignore list.
#
#   # Example for Subversion
#
#   $ svn mv config/database.yml config/database.example.yml
#   $ svn propset svn:ignore 'database.yml' config
#
#   # Example for Git
#
#   $ git mv config/database.yml config/database.example.yml
#   $ echo 'config/database.yml' >> .gitignore
#
#
# == Usage
#
# Include this file in your <tt>deploy.rb</tt> configuration file.
# Assuming you saved this recipe as capistrano_database_yml.rb:
#
#   require "capistrano_database_yml"
#
# Now, when <tt>deploy:setup</tt> is called, this script will automatically
# create the <tt>database.yml</tt> file in the shared folder.
# Each time you run a deploy, this script will also create a symlink
# from your application <tt>config/database.yml</tt> pointing to the shared configuration file.
#
# === Password prompt
#
# For security reasons, in the example below the password is not
# hard coded (or stored in a variable) but asked on setup.
# I don't like to store passwords in files under version control
# because they will live forever in your history.
# This is why I use the Capistrano::CLI utility.
#

unless Capistrano::Configuration.respond_to?(:instance)
  abort "This extension requires Capistrano 2"
end

Capistrano::Configuration.instance.load do

  namespace :deploy do

    namespace :db do

      desc <<-DESC
        Creates the database.yml configuration file in shared path.

        When this recipe is loaded, db:setup is automatically configured \
        to be invoked after deploy:setup. You can skip this task setting \
        the variable :skip_db_setup to true. This is especially useful \
        if you are using this recipe in combination with \
        capistrano-ext/multistaging to avoid multiple db:setup calls \
        when running deploy:setup for all stages one by one.
      DESC
      task :setup, :except => { :no_release => true } do

#        default_template = <<-EOF
#          base: &base
#            adapter: postgresql
#            username: #{Capistrano::CLI.ui.ask("Enter database username: ")}
#            password: #{Capistrano::CLI.password_prompt("Enter database password: ")}
#            pool: 5
#          development:
#            database: lahaswork_development
#            <<: *base
#          test:
#            database: lohaswork_test
#            <<: *base
#          production:
#            database: lohaswork_production
#            <<: *base
#        EOF

        default_template = <<-EOF
          base: &base
            adapter: mysql2
            encoding: utf8
            hostname: localhost
            username: #{Capistrano::CLI.ui.ask("Enter database username: ")}
            password: #{Capistrano::CLI.password_prompt("Enter database password: ")}
            pool: 5
          development:
            database: lahaswork_development
            <<: *base
          test:
            database: lohaswork_test
            <<: *base
          production:
            database: lohaswork_production
            <<: *base
        EOF

        location = fetch(:template_dir, "config/deploy") + '/database.yml.erb'
        template = File.file?(location) ? File.read(location) : default_template

        config = ERB.new(template)

        run "mkdir -p #{shared_path}/db"
        run "mkdir -p #{shared_path}/config"
        run "chmod g+rx,u+rwx #{shared_path}/config"
        put config.result(binding), "#{shared_path}/config/database.yml"
      end

      desc <<-DESC
        [internal] Updates the symlink for database.yml file to the just deployed release.
      DESC
      task :symlink, :except => { :no_release => true } do
        run "ln -nfs #{shared_path}/config/database.yml #{release_path}/config/database.yml"
      end

    end

    after "deploy:setup",           "deploy:db:setup"   unless fetch(:skip_db_setup, false)
    after "deploy:finalize_update", "deploy:db:symlink"

  end

end
```
#### config.ru
```
# This file is used by Rack-based servers to start the application.

require 'unicorn/oob_gc'
require 'unicorn/worker_killer'

#每10次请求，才执行一次GC
use Unicorn::OobGC, 10

#设定最大请求次数后自杀，避免禁止GC带来的内存泄漏（3072～4096之间随机，避免同时多个进程同时自杀，可以和下面的设定任选）
use Unicorn::WorkerKiller::MaxRequests, 3072, 4096

#设定达到最大内存后自杀，避免禁止GC带来的内存泄漏（192～256MB之间随机，避免同时多个进程同时自杀）
use Unicorn::WorkerKiller::Oom, (192*(1024**2)), (256*(1024**2))

require ::File.expand_path('../config/environment',  __FILE__)
run LohasWorkCom::Application
```
