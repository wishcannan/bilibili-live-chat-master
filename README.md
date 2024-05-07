
# Bilibili Live Chat

![Preview](https://i.loli.net/2020/06/20/vXuZKCq396co2HO.gif)

二次开发产物 没有来的即 布置反向代理服务 给https://blc.lolicon.app/ 用referer 扼住了咽喉 所以只能本地部署启动 完全没测试过开发平台 所以完全不推荐开发平台

## 食用步骤

1. 直接本地8080端口启动 就能使用 可太方便了



### 显示头像

自己写自己写 

## 隐私声明

本项目官方站点 [blc.lolicon.app](https://blc.lolicon.app/) 会额外使用到以下两个本人开源并部署在公共平台上的服务：

1. B站API反向代理服务 [Tsuk1ko/blc-proxy](https://github.com/Tsuk1ko/blc-proxy) 部署于 HuggingFace
2. B站扫码登录服务 [Tsuk1ko/bilibili-qr-login](https://github.com/Tsuk1ko/bilibili-qr-login) 部署于 HuggingFace

本站及上述服务不会收集任何信息，若不信任请勿在【关闭跨域模式】或【在普通连接模式下提供 cookie】的情况下使用本项目及【扫码登录】功能

## Project setup

```bash
yarn install
```

### Compiles and hot-reloads for development

```bash
yarn serve
```

### Compiles and minifies for production

```bash
yarn build
```

### Lints and fixes files

```bash
yarn lint
```

### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).
