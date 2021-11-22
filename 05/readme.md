# Alert报警
* 把报警自动发给指定的邮箱
## 配置邮箱
配置一个可以通过`Sentry`服务发送邮件的邮箱配置
* 修改文件
```
cd onpremise
vim ./sentry/config.yml
```
* 选择一个邮箱账号作为`Sentry`服务邮箱账号
  * 发件服务地址，例如qq邮箱：smtp.qq.com
  * 该邮箱账号
  * 该邮箱密码
* 将这些信息填写到`onpremise/sentry/config.yml`文件中
```
# mail.backend: 'smtp'  # Use dummy if you want to disable email entirely
mail.host: 'smtp.qq.com'
mail.port: 587 
mail.username: 'xxx@qq.com'
mail.password: 'xxx'
# mail.use-tls: false
# mail.use-ssl: false

# NOTE: The following 2 configs (mail.from and mail.list-namespace) are set
#       through SENTRY_MAIL_HOST in sentry.conf.py so remove those first if
#       you want your values in this file to be effective!


# The email address to send on behalf of
mail.from: 'xxx@qq.com'
```
* 停止`Sentry`服务
```
docker-compose down
```
* 启动`Sentry`服务
```
docker-compose up -d
```
* 发送测试邮件
![image](test-mail)
## 设置报警规则
可以设置一些报警规则，比如性能、错误，和对应的阈值。当触发报警规则后，会发送一封邮件到指定的邮箱，然后可以点击查看报警详情
![image](mail)