# rails + qq邮箱发邮件

背景：基于[rails 项目](./搭建后端项目.md)的基础上，搭建后端项目

**步骤**

1. ##### 生成邮件模板

```
    // 以ruby 3 版本为例
    bin/rails generate mailer User
```
2. ##### 配置邮件模板

`app/mailers/application_mailer.rb`

```ruby
    class ApplicationMailer < ActionMailer::Base
        default from: "email@qq.com"
        layout "mailer"
    end
```
`app/mailers/user_mailer.rb`

```ruby
    class UserMailer < ApplicationMailer
        def welcome_email(code)
            @code = code
            mail(to: "email@qq.com", subject: '番茄记账')
        end
    end
```

`app/views/user_mailer/welcome_email.html.erb`

```
    <!DOCTYPE html>
    <html>
    <head>
        <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
    </head>
    <body>
        你正在登录番茄记账，验证码是：<code><%= @code %></code>
    </body>
    </html>
```
3. ##### **配置邮件发送**

`config/environments/development.rb`

```ruby
    # Don't care if the mailer can't send.
    config.action_mailer.raise_delivery_errors = true
    config.action_mailer.perform_caching = false
    config.action_mailer.smtp_settings = {
        address:              'smtp.qq.com',
        port:                 587,
        domain:               'smtp.qq.com',
        user_name:            'mail@qq.com',
        password:             Rails.application.credentials.email_password,
        authentication:       'plain',
        enable_starttls_auto: true,
        open_timeout:         10,
        read_timeout:         10
    }
```
4. ##### 配置qq邮箱服务

登录qq邮箱，在设置->账号中开启`POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务`，根据提示发送验证码，再复制**授权码**。

**创建开发环境的秘钥**

运行如下命令

```
    // 管理授权码，仅在编辑模式下可见
    EDITOR="code --wait" bin/rails credentials:edit
```
如上命令所做的事情：

1.  会生成`config/credentials.yml.enc`（加密后的文件）和`config/master.key`（秘钥）两个文件；
2. 使用vscode 编辑器打开一个临时文件，会有一个字段`secret_key_base`，值是一个随机生成的字符串，长度大概有128位，可以用来加密session等其他加密功能；
3. 用户在打开的临时文件中添加如下内容`email_password: "授权码"`，关闭文件，保存即可；

**注意**

1. 若多人共享master.key，则`config/credentials.yml.enc`、`config/master.key`不被git ignore;
2. 若不共享，则每个人创建自己的master.key，`config/credentials.yml.enc`、`config/master.key`被git ignore;；

若要查看秘钥内容，可执行如下命令：

```
    bin/rails c
    // 查看所有配置
    Rails.application.credentials.config
```

若要修改秘钥内容，执行如下命令

```
	EDITOR="code --wait" bin/rails credentials:edit
```

**创建生产环境的秘钥**

运行如下命令

``````
    // 管理授权码，仅在编辑模式下可见。生产环境
    EDITOR="code --wait" bin/rails credentials:edit --environment production
``````

如上命令所做的事情：

1. 会生成`config/credentials/production.yml.enc`（加密后的文件）和`config/credentials/production.key`（秘钥）两个文件；
2. 使用vscode 编辑器打开一个临时文件，会有一个字段`secret_key_base`，值是一个随机生成的字符串，长度大概有128位，可以用来加密session等其他加密功能；
3. 用户在打开的临时文件中添加如下内容`email_password: "授权码"`，关闭文件，保存即可；

**注意**

1. 秘钥文件不要传到git仓库中，在`.gitignore`文件中添加这一行`/config/credentials/production.key`；
2. `config/credentials/production.yml.enc`不被git ignore
3. 读取key们的代码与开发环境一致；
4. 秘钥需要放到生产环境中，将内容写到环境变量，并做好备份；

若要查看秘钥内容，可执行如下命令：

```
    RAILS_ENV=production bin/rails c
    // 查看所有配置
    Rails.application.credentials.config
```

若要修改秘钥内容，执行如下命令

``````
    EDITOR="code --wait" bin/rails credentials:edit --environment production
``````

### 验证邮件发送是否成功

打开控制台，运行如下命令

```
    bin/rails console
    UserMailer.welcome_email("123").deliver
```
即可收到邮件。
