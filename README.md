<h1 align="center">餐厅管理系统</h1>

## 需求分析

用户在生活中可能有朋友介绍、或者在网上发现一些感兴趣的餐厅，希望以后某天前往尝试，但是可能忘记，而记录在备忘录中可能比较乱，所以设计了这个餐厅管理系统。

## 概要设计

这个数据库应该具备如下几个字段：

- **餐厅名称**：餐厅的名称
- **联系方式**：用于多人就餐等场景时的预约
- **餐厅地址**：餐厅的地址

并且应该隔离不同用户存储的数据。因此需要基本的用户管理：

- **用户登录**
- **用户注册**
- **修改名字或密码**

## 详细设计

### 概要

应用程序由 [JonathanSilver/bserv](https://github.com/jonathansilver/bserv) 后端、 Vue 前端和 Postgres 数据库三部分组成。

### 数据库

数据库中有两个表，如下：

```sql
CREATE TABLE auth_user (
    id serial PRIMARY KEY,                               -- 用户的编号，是用户的唯一标识
    name character varying(255) NOT NULL,                -- 用户的姓名，用于显示在 UI 上
    password character varying(255) NOT NULL,            -- 用户的密码 hash 值
    email character varying(255) NOT NULL UNIQUE         -- 用户的邮箱，是候选键
);

CREATE TABLE restaurants (
    id serial PRIMARY KEY,                               -- 餐厅的编号，是餐厅的唯一标识
    user_id integer references auth_user,                -- 该记录的所有者的编号
    name character varying(255) NOT NULL,                -- 餐厅名称
    address character varying(255) NOT NULL,             -- 餐厅地址
    contact character varying(255) NOT NULL              -- 联系方式
);
```

### 认证与通信

为了保证传输安全，在登录或注册时所有输入的密码都会在前端直接 hash 之后再传输。

后端是一个无状态的服务，当涉及与用户相关的操作时，前端通过在 POST 请求的请求体中传输用户邮箱以及（hash过的）密码来向后端提供认证信息。

前端和后端通过 HTTP 请求通信。为了避免跨域问题，前后端统一由 Express 中间件代理，`/api` 相关路径会被转发到后端。

### 前端 [arielherself/db-pj1-frontend](https://github.com/arielherself/db-pj1-frontend)

前端使用 Vue 和 Vuetify 组件库实现，提供了稳定的后端功能的呈现。同时对于一些常见的用户习惯进行了优化，例如按 Enter 键提交表单。

### 后端 [arielherself/db-pj1-frontend](https://github.com/arielherself/db-pj1-backend)

在 bserv 项目的基础上实现了以下功能：

- 用户登录
- 用户注册
- 修改用户信息
- 查询某位用户添加的所有餐厅
- 添加餐厅
- 修改餐厅信息
- 删除餐厅
- 搜索某位用户的、带特定关键词的餐厅

### 容器化

由于本项目涉及诸多配置依赖以及不同的部署方法，应当进行容器化以简化部署流程。

- Boost 库容器：由于 Boost 库体量较大，所以按照 bserv 项目的说明将 Boost 库自动化并将编译好的库封装在容器 [arielherself/build-boost](https://hub.docker.com/r/arielherself/build-boost) 中
- 后端容器：基于 [arielherself/build-boost](https://hub.docker.com/r/arielherself/build-boost) 编译 bserv 及其相关依赖，封装设计好的配置文件在 [arielherself/bserv_backend](https://hub.docker.com/r/arielherself/bserv_backend)
- 前端容器：将前端的 Express.js 脚本和 Vue 前端封装在 [arielherself/bserv_frontend](https://hub.docker.com/r/arielherself/bserv_frontend)

## 部署方法

克隆本仓库，然后执行命令：

```bash
docker compose up
```

即可在 http://localhost:8080 访问应用。如果需要在其他端口部署服务，请在 `docker-compose.yml` 中修改 `frontend` 的端口映射。
