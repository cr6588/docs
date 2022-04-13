---
title: "Nvm"
date: 2022-04-06T21:43:22+08:00
draft: true
---

nvm是管理node工具，可以安装切换多版本node
#使用国内源安装node
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
#先卸载cnpm
npm remove -g cnpm --registry=https://registry.npmmirror.com
#安装最新的lts版本
nvm install --lts
nvm use --lts
npm install -g cnpm --registry=https://registry.npmmirror.com
nvm alias default node
