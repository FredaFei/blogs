## rails + qq邮箱发邮件

背景：基于[rails 项目](./搭建后端项目.md)的基础上，搭建后端项目

### 步骤

1. 生成邮件模板

```
    // 以ruby 3 版本为例
    bin/rails generate mailer User
```
2. 配置邮件模板

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
3. 配置邮件发送

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
4. 配置qq邮箱服务

登录qq邮箱，在设置->账号中开启`POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务`，根据提示发送验证码，再复制授权码，运行如下命令
```
    // 管理授权码，仅在编辑模式下可见
    EDITOR="code --wait" bin/rails credentials:edit
```
将打开一个临时文件，在文件中添加如下内容`email_password: "授权码"`，关闭文件，保存即可。

### 验证邮件发送是否成功

打开控制台，运行如下命令

```
    bin/rails console
    UserMailer.welcome_email("123").deliver
```
即可收到邮件。
