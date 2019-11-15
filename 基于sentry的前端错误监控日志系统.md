# 基于sentry的前端错误监控日志系统

部署sentry服务器/前端项目部署

# 背景

---

- 线上环境报错问题难以定位
- 开发人员无法第一时间发现问题
- 有些问题就是需要在特定环境下才会触发，QA不一定能覆盖
- 官方服务超过一定量需要收费&安全问题 → 自己搭建

# 环境部署

---

### 1. Docker部署

[http://www.runoob.com/docker/macos-docker-install.html](http://www.runoob.com/docker/macos-docker-install.html)

tip：mac 版的安装，docker-compose 已经一起安装好了

## 2. Sentry部署

1. git clone [https://github.com/getsentry/onpremise.git](https://github.com/getsentry/onpremise.git)
2. 按照项目的 [readme.md](http://readme.md/) 开始依照步骤搭建 

    1. `docker volume create --name=sentry-data && docker volume create --name=sentry-postgres` - Make our local database and sentry volumes
        Docker volumes have to be created manually, as they are declared as external to be more durable.
    2. `cp -n .env.example .env` - create env config file
    3. `docker-compose build` - Build and tag the Docker services
    4. `docker-compose run --rm web config generate-secret-key` - Generate a secret key.
        Add it to `.env` as `SENTRY_SECRET_KEY`.
    5. `docker-compose run --rm web upgrade` - Build the database.
        Use the interactive prompts to create a user account.
    6. `docker-compose up -d` - Lift all services (detached/background mode).
    7. Access your instance at `localhost:9000`!

 3.  创建项目

## 3. Sentry 界面介绍

![](/post-img/sentry/img01.png)

- Organization(组织)：Sentry项目中用于管理的最大单位，比如某某产品的技术组
- Teams(团队)：从属于组织，一个组织下可以有多个团队，比如前端团队
- Members(成员)：团队的成员，以邮箱账户为单位，成员的可以加入多个团队，
- Projects(项目)：错误信息所归属的最小单位，一个团队下的成员拥有该团队下所有项目的查看权限

## 4. 应用

3.1 项目创建

- Node

    var Raven = require('raven')
    
    Raven.config('http://5fba76873bb74bb3b65707b60ec69eca@localhost:9000/2').install()
    
    app.use(Raven.requestHandler()) // Sentry必须在所有中间件之前
    /// ////// 加载中间件 //////////////
    app.use(Raven.errorHandler()) // Sentry错误中间件必须在所有error中间件之前
    /// ////// 加载error中间件 //////////////

- React

    <script src="https://cdn.ravenjs.com/3.26.2/raven.min.js" crossorigin="anonymous"></script>
    
    Raven.config('http://95eba071ce8f4abeb1911a260aeec071@localhost:9000/3', {
      release: 'dev@1.0.0'
    }).install()

3.2 异常捕获

    // 无法捕获异步错误（比如定时器、接口请求错误） raven.caputureException() 进行主动上报
    // 主动抛出的异常时使用new error进行创建，这样能更好的定位异常所在位置
    Raven.captureException(new Error('emailTest'))
    
    // 只发送消息
    Raven.captureMessage('Something went wrong')

3.3 sourcemap 配置，只为上传Sentry，禁止上传到生产环境

- 获取认证的 token：API Keys → Auth Tokens
- 配置文件 .sentryclirc
- 配置 webpack SentryPlugin

3.4  邮件报警，可自定义报警时间间隔，邮件模版

- onpremise根目录里配置config.yml
- Admin → Mail 测试邮件发送
- project → setting → Alerts

### 5. 注意点

开启两步验证 Authy

删除某个release时需要将其下的异常处理掉,并将该版本的sourcemap文件清空,完成上面两步可能还要等待2小时才能删除，不然会报错：该版本还有其它依赖。

- [ ]  Gitlab or JIRA Issue 关联
- [ ]  周报：有设置选项但本地设置失败

![](https://discourse-cdn-sjc2.com/standard10/uploads/sentry/original/1X/1a507e368879cbce1a34c1ca2501c0d32742135b.png)

reference link：

[https://www.cnblogs.com/xakoy/p/9636393.html](https://www.cnblogs.com/xakoy/p/9636393.html)

[https://juejin.im/post/5c063854e51d451df309410b](https://juejin.im/post/5c063854e51d451df309410b)

[https://zhuanlan.zhihu.com/p/51446011](https://zhuanlan.zhihu.com/p/51446011)

[https://blog.csdn.net/m18510011124/article/details/80935343](https://blog.csdn.net/m18510011124/article/details/80935343)

[https://juejin.im/post/5b226cbe51882574d02f9f62](https://juejin.im/post/5b226cbe51882574d02f9f62)

[https://icewind-blog.com/2018/12/11/integrate-sentry-with-javascript/](https://icewind-blog.com/2018/12/11/integrate-sentry-with-javascript/)

[https://juejin.im/post/5c9f4639e51d452724598c07](https://juejin.im/post/5c9f4639e51d452724598c07)

上传sourceMap：sentry-cli releases -o zati-sentry -p za-anan-life-pc-react files prd@1.0.0 upload-sourcemaps ~/Downloads/temp --url-prefix ~/
