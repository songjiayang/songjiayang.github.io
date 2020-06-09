---
title: "repo_url changes but not work in capistrano3"
date: 2015-07-29 22:31:00 +0800
tags: [ruby]
---

<div style='text-align:center;'>
  <img src='/images/capistrano.jpg'/>
</div>

背景： 项目使用了 capistrano（3.4.0）的  git 部署方式； 但对于已有部署项目，当修改配置 repo_url 后，重新执行部署任务,
服务器依然从旧仓库地址拉取代码。

登录服务器，查看 git 配置文件（PROJECT/repo/config），依然是旧版本信息。因为每次执行 deploy 的时候，git 仓库信息都是从这个文件读取出来的，解决办法就是修改这个文件。

这里提供 3 种解决方案：

1. 删除 PROJECT/repo 文件夹，当执行部署任务，如果此目录不存在， capistrano 会自动创建，当然使用最新配置。
2. 手动修改 PROJECT/repo/config 文件内容，主要针对 `url = git@git xx` 这行。  
3. 在项目中新建一个 rake task，代码大概如下：

```

#lib/capistrano/tasks/git.rake
namespace :git do
  desc 'change the cached repo_url if repo_url changes in config'
  task :update_repo_url do
    on release_roles :all do
      with fetch(:git_environmental_variables) do
        within repo_path do
          current_repo_url = execute :git, :config, :'--get', :'remote.origin.url'
          unless repo_url == current_repo_url
            execute :git, :remote, :'set-url', 'origin', repo_url
            execute :git, :remote, :update

            execute :git, :config, :'--get', :'remote.origin.url'
          end
        end
      end
    end
  end
end
```  
当修改了 repo_url，在本地执行 `cap production git:update_repo_url` 即可。

以上 3 种方法没有优劣之分，毕竟修改 repo_url 是一个非常低频的操作。
