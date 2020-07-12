---
title: CI/CD
date: 2019-03-11 11:30:45
updated: 2020-07-12 12:00:00
tags: [DevOps]
---

Continuous integration

<!-- more -->

## General Flow

编写代码、单元测试

推送代码、触发CI流程

编译、跑单元测试、覆盖率、静态检测

满足条件合入开发分支、触发二进制打包

二进制包推送至软件仓库、触发自动部署脚本

测试环境从软件仓库拉取安装包或者镜像

定期运行功能测试套件、并生成测试报告

## Tools

Jenkins、GitLab、Kubernetes、Docker

## Tests

UT、FT、ST

### 单元测试

重点关注覆盖率

### 功能测试

以单个接口、业务流程测试为主。

围绕功能、用户故事编写测试用例，生成可读测试报告。