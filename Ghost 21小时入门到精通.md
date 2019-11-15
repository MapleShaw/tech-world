# Ghost 21小时入门到精通

The style guidelines and best practices for our engineering team.

# 安装Ghost

---

- 全局安装 Ghost-CLI进行本地安装
- Git源码安装

    npm install ghost-cli -g
    // Ghost-CLI 安装完成后，cd 到一个空目录下安装 Ghost，该目录就是 Ghost 的安装目录。
    ghost install local

前台：[http://localhost:2368](http://localhost:2368/)

后台：[http://localhost:2368/ghost](http://localhost:2368/ghost)

# Ghost管理后台

---

- 基本设置
- 主题
- 多语言
- 代码注入
- 插件

# Ghost常用命令

---

[常用命令](https://www.notion.so/d8e273522fb246b196b219672190c729)

# Handlebars expressions

---

[Handlebars.js: Minimal Templating on Steroids](https://handlebarsjs.com/expressions.html)

# Ghost部署

---

### 测试服务器部署

[记一次CentOS上安装Ghost](https://freekingd.com/ghost-install/)

    // 官方建议使用非root用户安装和启动Ghost
    1. npm install -g n
    2. n 10.13.0
    3. ghost install local
    4. config.development.json配置0.0.0.0

### 生产环境多点部署

- sqlite > mysql 发布文章，更新主题会出现访问不到
- 数据库不稳定，一些数据丢失（author下的url）
- 邀请新人的发送邮箱，默认生成的[ghost@home-test.zatech.com](mailto:ghost@home-test.zatech.com)，会被公司网络block

# Ghost 对外 API

---

- 获取特定文章的接口

[Javascript Content API Client Library - Ghost](https://docs.ghost.org/api/javascript/content/)

# Ghost项目代码详解

---

- 配置文件 config.development.json
- /themes 文件夹