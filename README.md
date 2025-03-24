# **EasyPan** 网盘项目

**EasyPan** 是一个基于 **Spring Boot** 和 **Vue 3** 的网盘项目，旨在帮助用户上传、下载和分享文件。

该项目提供了一个方便快捷的文件管理平台，让用户能够轻松地存储和访问他们的文件。

---

## **技术栈**

- **后端**：Java Spring Boot, Spring Security, Spring Data JPA, MySQL, Redis, FFmpeg
- **前端**：Vue 3, Vue Router, Vuex, Axios, Element Plus, Highlight.js
- **其他**：Node.js, Webpack, Babel, Sass, SparkMD5, Vue Cookies, Vue Clipboard3, Vue PDF Embed, XLSX, DPlayer, APlayer, docx-preview

---

## **环境要求**

在开始之前，请确保以下环境已经安装：

- **Java JDK** 8 或以上版本
- **Node.js** (建议使用 v16 或以上版本)
- **MySQL** 数据库
- **Redis**：用于缓存和 Session 管理
- **FFmpeg**：处理视频和音频文件
- **Nginx**：用于前端部署和反向代理
- **SwitchHosts**：配置 host 方便开发时的本地域名映射

---

## **后端项目创建和配置**

在开始前，请确保你已经安装了 Java JDK 8 或以上版本，并且配置了 Maven 环境。

1. 配置名为 **"easypan"** 的数据库，使用 **Navicat** 连接到 **MySQL** 数据库，方便查看和管理数据库内容。

---

## **前端项目部署和配置**

在开始前，请确保你已经安装了 **Node.js** 和 **npm**。

1. 将 **"dist"** 目录下的文件部署进 **Nginx** 中。

---

## **使用 QQ 互联实现注册和登录**

EasyPan 项目使用了 **QQ 互联** 作为第三方登录认证方式。为了在你的项目中实现此功能，你需要进行以下配置：

1. **获取 QQ 互联的 AppID 和 AppKey**
    - 前往 **QQ互联开放平台** 注册一个应用，获取对应的 **AppID** 和 **AppKey**。

2. **配置后端的 QQ 互联信息**

---

## **EasyPan 优点**

- **简洁易用**：EasyPan 拥有直观的用户界面和用户友好的操作，无论是上传、下载还是分享文件，都能够在几步之内完成。

- **多媒体支持**：通过集成 **FFmpeg**、**DPlayer** 和 **APlayer** 等工具，EasyPan 支持视频、音频文件的在线播放，让用户可以在线观看或收听媒体文件。

- **QQ互联登录**：EasyPan 实现了使用 QQ 互联进行注册和登录的功能，用户可以方便地通过 QQ 账号登录，避免繁琐的注册流程。

- **数据安全保障**：EasyPan 采用了 **Spring Security** 进行用户认证和授权，通过 **Redis** 进行数据缓存，保障用户数据的安全和高效访问。

- **扩展性**：EasyPan 的后端使用 **Spring Boot** 框架，前端使用 **Vue 3** 框架，代码结构清晰，易于维护和扩展，方便添加更多功能。

---

## **联系方式**

- **邮箱**：[961116028@qq.com](mailto:961116028@qq.com)

---

### 觉得好用就奖励一下作者吧！！！

![二维码](https://web-work-bianxin.oss-cn-hangzhou.aliyuncs.com/42e775d5d6c2483a16d0b95adf24e01.jpg)

---

**EasyPan** 旨在为用户提供一个简单、高效的网盘服务，让用户能够轻松管理和共享自己的文件。无论是个人使用还是团队协作，EasyPan 都能够满足你的需求，让文件管理变得更加便捷和愉快。

---

这样排版之后，整个文档看起来会更加有条理，而且容易阅读。希望这符合您的需求！