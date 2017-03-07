---
title: 前端开发大纲
date: 2017-3-7 11:32:00
tags:
  - fe
  - dev
---

### 项目结构

为了更好地协作，项目结构最好统一起来。一般可以包括如下目录结构：

``` 
▸ dist/          # 纯静态编译后的代码
▸ lib/           # 服务端可直接运行的代码，或作为依赖存在的代码
▸ scripts/       # 辅助用的脚本，比如编译、部署、配置管理
▸ src/           # 需要编译的源码，如ES6，JSX
  .editorconfig
  .gitignore
  config.yml     # 存放一些敏感的配置，与nconf配合使用（见后）
  process.yml    # PM2启动服务端用的配置文件
  README.md      # 介绍项目的基本信息，和常用的命令，方便其他协作者上手
```

注意事项：
- 项目文件命名避免使用大写字母，最好使用全小写加连字符的形式，如`my-project`，以免在不同平台间出现问题。
- `config.yml`中的信息敏感，一定要记得将`config.yml`加入`.gitignore`，或者直接使用环境变量替代。

### 自动化脚本

每个项目最好都可以通过最简单的命令来完成一些常用操作，如：

- `npm run dev`：一键开发
- `npm run build`：一键编译
- `npm run deploy`：一键部署
- `npm start`：服务端项目一键启动

然后通过`NODE_ENV`环境变量控制操作的类型，如：

``` sh
# 编译用于测试环境
$ npm run build

# 编译用于生产环境
$ npm run build --production
```

一键部署到生产环境需要慎重使用，避免误操作，可以加入显眼的提示和延时等。

### 配置管理

配置管理推荐使用[nconf](https://github.com/indexzero/nconf)，它是一个分层的配置管理工具，可以依次从每个层去查找相应的配置，轻松地实现配置的覆盖，而且支持`yaml`语法。

举个栗子：

``` js
const nconf = require('nconf');

nconf

// 如果需要强制覆盖一些配置，可以用overrides写到最前面一层
.overrides({
  weather: 'sunny',
})

// 从配置文件读取，如果文件不存在则忽略
.file({
  file: 'config.yml',
  format: require('nconf-yaml'),    // 如果不指定format则默认json
})

// 从环境变量读取
.env()

// 从运行时传进来的参数读取
.argv()

// 默认配置
.defaults({
  weather: 'cloudy',
  NODE_ENV: 'development',
})

// 还可以强行修改当前的配置，优先级最高
nconf.set('weather', 'rainy');

// 获取配置
// console.log(nconf.get('weather'));

module.exports = nconf;
```

`nconf.get`会从每一层配置依次查找，返回第一个找到的结果。

在上面的例子中，`NODE_ENV`被默认设置为`development`，如果在环境变量中设置`NODE_ENV=production`，则`nconf.get('NODE_ENV') === 'production'`，因为`.env()`这一层在`.defaults({...})`这一层的前面，会先被找到。

### Git的使用

1. 控制好提交的粒度，[写好提交信息](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)，方便回滚。
1. 打好标签（见后），方便回滚。
1. 合并分支前先确保更新到最新状态，避免旧代码覆盖掉新代码，同时避免一个开发分支跨越主分支上很多提交、甚至版本号，方便回滚。
1. 多人协作时，在不同分支上进行开发，避免在一个分支上修改另一个分支里开发的内容造成冲突。
1. 分支命名最好让其他人一眼明白其用处，如`fix/awkward-bug`、`feature/awesome-feature`。

### 标签的使用

标签相当于一个不活跃的分支，其好处是一直存在，可以在需要的时候快速找到。

我们应该在每次部署是都打一个标签，这样如果之后出了问题，要回滚到上一个版本，只需要找到上一个版本对应的标签就好了。打标签可以使用命令：

``` sh
$ npm version [ patch | minor | major | <version_number> ]
```

### 服务端管理

1. 纯静态项目

   纯静态项目最好的方案是部署到CDN上，但是有时候涉及缓存，或者是URL地址的原因，我们会在自己的服务器上来serve。最理想的方案是用nginx直接指向静态文件夹。

   - 生产环境：nginx直接serve静态文件

     ``` conf
     server {
       listen 4000;
       server_name single.dog;

       location / {
         root /home/hugo/web/single_dog;

         # 如果是单页面应用还需加上
         try_files $uri /index.html;
         add_header Cache-Control no-cache;
       }
     }
     ```

     如果配置好后任何地址都出现500，则多半是nginx没有权限访问静态文件夹（可以通过查看日志明确），可以通过以下命令验证：

     ``` sh
     $ sudo -u nginx stat /home/hugo/web/single_dog
     # 出错则说明没有权限
     ```

     添加权限的时候，需要给目标目录及其所有上级目录都加上其他用户（Other）的执行权限（x）：

     ``` sh
     $ sudo chmod o+x /home /home/hugo /home/hugo/web /home/hugo/web/single_dog
     ```

  - 测试环境：可以用nginx，也可以用[spa-static](https://github.com/gera2ld/spa-static)

1. Node.js服务项目

   项目通过`process.yml`配置好以后，直接用[PM2](http://pm2.keymetrics.io/)管理。

   ``` sh
   # 启动项目
   $ pm2 start process.yml

   # 重启项目
   $ pm2 restart process.yml

   # 列出所有项目
   $ pm2 ls

   # 查看日志
   $ pm2 logs <id_or_name>
   ```

   注：启动项目后请一定查看日志或者项目状态，确定项目正常运行。
