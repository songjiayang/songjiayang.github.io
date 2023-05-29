---
title: "Rails deploy with capistrano3"
date: 2014-11-20 22:31:00 +0800
tags: [ruby]
---

今天将公司 [Reset](http://gigareset.org/locations/33) 项目的部署由
capistrano2+unicorn+sidekiq 换成了 capistrano3+puma+sidekiq 。

以下是个人整理的几个重要地方。

##### 1. 修改 Gemfile 文件
```
#Gemfile
gem 'puma'
group :development do
  .....
  # cap deploy plugins
  gem 'capistrano', '~> 3.2.0'
  gem 'capistrano-rvm', '~> 0.1.2'
  gem 'capistrano-bundler', '~> 1.1.3'
  gem 'capistrano-rails', '~> 1.1.2'
  gem 'capistrano-sidekiq', '~> 0.4.0'
  gem 'capistrano3-puma', '~> 0.8.3'
  .....
end
```

##### 2. 修改 Capfile 文件

```
#Capfile

# Load DSL and Setup Up Stages
require 'capistrano/setup'

# Includes default deployment tasks
require 'capistrano/deploy'

# Includes tasks from other gems included in your Gemfile
#
# For documentation on these, see for example:
#
#   https://github.com/capistrano/rvm
#   https://github.com/capistrano/bundler
#   https://github.com/capistrano/rails
#   https://github.com/seuros/capistrano-sidekiq
#   https://github.com/seuros/capistrano-puma

require 'capistrano/rvm'
require 'capistrano/bundler'
require 'capistrano/rails'
require 'capistrano/sidekiq'
require 'capistrano/puma'

# Loads custom tasks from `lib/capistrano/tasks' if you have any defined.
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }

```

##### 3. 修改 deploy.rb 文件
```
#config/deploy.rb

# Ensure that bundle is used for rake tasks

lock '3.2.1'

set :application, 'gigareset'
set :repo_url, 'git@github.com:rdd-giga/reset.git'

set :scm, :git
set :user, :deploy
set :use_sudo, false
set :git_enable_submodules, true

set :deploy_to, '/var/www/gigareset'

set :linked_files, %w{config/database.yml}
set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

set :ssh_options, {
  forward_agent: true,
  port: 22,
  keys: [File.join(ENV["HOME"], ".ssh", "id_rsa")]
}

set :keep_releases, 5

```
##### 4. 修改 deploy/production.rb 文件
```
#config/deploy/production.rb

set :stage, :production
set :branch, "master"

server 'gigareset.org', user: 'deploy', roles: %w{web app db}

```

最后在服务器上创建好需要shared的文件，执行 `cap production deploy` 皆可。
以上配置属于官方默认配置，如果需要更多配置，请参考对应 gem 官方文档或查看源码。
