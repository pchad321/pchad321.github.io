---
title: HEXO安装与使用 
date: 
modified:
categories:
- 前端
tags:
- HEXO
- Linux 
---

### nodejs安装

> 通过EPEL的方式进行安装

```
# 安装epel
yum install -y epel-release
yum repolist

# 安装node
yum install -y nodejs
node --version
yum install npm
```

### hexo安装

```
npm install hexo-cli -g
```

> 快速新建一个博客系统

```
hexo init blog
cd blog
npm install
hexo server
```

### hexo常用命令

```
# 清空缓存
hexo clean
# 生成静态网页
hexo g
# 启动hexo服务，默认端口为4000
hexo s
```

### 安装NEXT主题
> 资源链接

https://github.com/iissnan/hexo-theme-next

> 安装方法

```
$ cd hexo
$ ls
_config.yml  node_modules  package.json  public  scaffolds  source  themes
$ mkdir themes/next
$ curl -s https://api.github.com/repos/iissnan/hexo-theme-next/releases/latest | grep tarball_url | cut -d '"' -f 4 | wget -i - -O- | tar -zx -C themes/next --strip-components=1
```

### NEXT的相关配置

> 对于整个项目，有一个站点配置文件`_config.yml`，该文件位于根目录下，在本项目中位于`/root/blog`目录下，对于每个主题，都有一个主题的配置文件`_config.yml`，位于每个主题的根目录下，在本项目中位于`/root/blog/themes/next`目录下：

```
$ ls /root/blog/
_config.yml  node_modules  public     source
db.json      package.json  scaffolds  themes

$ ls /root/blog/themes/next/
bower.json   gulpfile.coffee  layout   package.json  README.md  source
_config.yml  languages        LICENSE  README.cn.md  scripts    test
```




### 添加分类标签

> 如果需要给首页添加分页，首先修改主题配置文件，将menu下的about前的#去除：

```
menu:
  home: /
  #tags: /tags/ || tags
  categories: /categories/
  archives: /archives/
  about: /about/
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

> 然后在source目录下新建一个categories文件夹，并在该文件夹中添加一个index.md文件

```
$ cd /root/blog/source
$ mkdir categories/
$ vim index.md
```

> 在文件中添加以下内容

```
---
title: 文章分类
date: 日期
type: "categories"
comments: false
---
```

> 然后在文件的头部添加以下内容：

```
---
title: Hello World
categories: 
- test
---
```

### 添加动态背景

> 打开  next/layout/_layout.swig
在`</body>`之前添加代码(注意不要放在`</head>`的后面)：

```
{% if theme.canvas_nest %}
<script type="text/javascript" src="//cdn.bootcss.com/canvas-nest.js/1.0.0/canvas-nest.min.js"></script>
{% endif %}
```

> 修改主题配置文件，将canvas_nest修改为true

```
canvas_nest: true
```

### 添加更新时间

> 修改语言配置文件`/themes/next/languages/zh_Hans.yml`，在post下添加以下内容：

```
post:
  updated: 更新于
```

> 修改主题配置文件`/themes/next/_config.yml`

```
updated_at: true
```

> 写文章的时候可以直接在文章开头设置更新时间

```
modified:
```

### 添加站内搜索
#### 安装hexo-generator-search

> 在站点的根目录下执行以下命令：

```
npm install hexo-generator-search --save
```

#### 安装 hexo-generator-searchdb

> 在站点的根目录下执行以下命令：

```
npm install hexo-generator-searchdb --save
```

#### 启用搜索

> 编辑站点配置文件，新增以下内容到最后：

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

> 编辑主题配置文件，将local_search下的enable改为true：

```
local_search:
  enable: true
```

### 上传代码到github

#### 注册Github的账号

略

#### 创建Repository
创建的时候需要注意Repository的名字。例如我的Github账号是pchad321，那么应该创建的Repository的名字为：pchad321.github.io

#### 修改配置文件

1. 进入刚刚创建的Repository，复制Repository的连接，例如https://github.com/pchad321/pchad321.github.io.git

2. 修改站点配置文件

  ```
  deploy:
    type: git
    repo: https://github.com/pchad321/pchad321.github.io.git
    branch: master
    message: 'updated at:{{now("YYYY-MM-DD HH/mm/ss")}}'

  ```

#### 设置SSH keys

1. 配置本机全局git环境

  ```
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
  ```

2. 生成SSH密钥

  ```
  # -C后面是github上的用户邮箱地址
  ssh-keygen -t rsa -C you@126.com

  # 然后一路回车
  ```

3. 将SSH密钥添加到github中

  ```
  less ~/.ssh/id_rsa.pub
  ```

  > 复制出现的一堆密码，然后在github的SSH and GPG keys中粘贴这一段密码

4. 设置的验证

  在本地输入以下代码：

  ```
  ssh -T git@github.com
  ```

  > 如果在最后出现了你配置的用户名，说明配置成功

5. 提交代码到github上

  ```
  hexo generate
  hexo deploy
  ```

  > 在提交代码时可能会出现`ERROR Deployer not found: git`的错误，此时只需要运行以下命令

  ```
  npm install --save hexo-deployer-git
  ```
