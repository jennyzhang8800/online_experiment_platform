参照[官网](https://about.gitlab.com/downloads/)的安装步骤

git_config.backup目录中为Gitlab服务器上的配置文件备份

安装完成后,做如下修改:

[user.rb备份](https://github.com/rainymoon911/online_experiment_platform/blob/master/GitLab/git_config.backup/users.rb)

1.去除邮件激活


    vi ./embedded/service/gitlab-rails/lib/api/users.rb
    
在create user函数中加入如下语句
    
    user.skip_confirmation!
    if user.save
    ...

[gitlab.rb备份](https://github.com/rainymoon911/online_experiment_platform/blob/master/GitLab/git_config.backup/gitlab.rb)

2.去除前台的注册功能(默认是关闭的,若开启,按下列步骤关闭)

    vi /etc/gitlab/gitlab.rb
    gitlab_rails['gitlab_signup_enabled'] = false
    //使配置生效
    sudo gitlab-ctl reconfigure

3.配置不允许用户修改邮件:

    vi /etc/gitlab/gitlab.rb
    gitlab_rails['gitlab_username_changing_enabled'] = false
    //使配置生效
    sudo gitlab-ctl reconfigure
    
4.去除邮件查重(由于shibboleth登录时会报邮件已存在的错误,不像OpenEdX能与本地账号绑定)

5.创建教师账号,用户名teacher,教师以管理员的账号在Docker的配置文件中会用到
