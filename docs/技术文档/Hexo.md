# Hexo

Hexo 是一个快速、简洁且高效的静态博客框架，它基于 Node.js 运行，可以将我们撰写的 Markdown 文档快速解析渲染成静态的 HTML 网页。

[Hexo](https://hexo.io/zh-cn/)

[Hexo+Github搭建免费博客｜基础教程(一)：全网最细致全面的教程](https://sspai.com/post/84847)

[使用GitHub部署静态博客HEXO](https://tech.yemengstar.com/hexo-tutorial-deploy-githubpages-beginner/)

## Github Page or Gitee Page?

- Gitee Page：

    1 Gitee Page 的开通需要身份证认证

    2 Gitee是有5G空间限制的

    3 部署博客后需要在Gitee仓库手动更新，比较要命

- Github Page：

    只需要梯子，其他都优势于Gitee

### 准备GitHub Page

- 创建仓库，请使用`<username>.github.io`命名你的仓库

    hexo部署后，如果deploy bramch分支与仓库不同，需要配置仓库page指定分支

### 一、Hexo环境配置搭建

- 1.安装Git

    验证cmd：`git --version`

- 2.安装Nodjs

    [Nodjs](http://nodejs.cn/download/)

    验证cmd：`npm version`

### 二、Hexo本地创建博客

- npm安装hexo

    安装cmd：`npm install hexo-cli -g`

    验证cmd：`hexo -version`

- hexo初始化：

    方法1.创建一个文件夹，例如../blog，命令行(git)在该路经下使用 `hexo init`

    方法2.`hexo init blog`，这会在在当前命令行(git)路径下创建一个blog文件夹

- 安装组件：

    在blog路径，命令行(git) `npm install`

### 三、生成博客，预览博客，部署博客

- 预览页面(hexo server)：$ hexo s

    将生成本地连接`http://localhost:4000`来预览
    ***这是本地的实时预览，也就是本地的任何修改，都可以马上在这里看见预览效果***

- 生成页面(hexo generate)：$ hexo g
  
    生成部署页面 .deploy.git 再通过部署推送到github托管

- 部署页面(hexo deploy):$ hexo d

    需要配置好_config.yml下的deploy，来部署博客，例如

        deploy:
            type: git
            repo: git@github.com:Xuser7/Xuser7.github.io.git
            branch: main

    需要确保git ssh可以连通哦，才能部署成功

### 四、配置个人博客

以下举例以 \blog 表示博客文件夹

#### 配置主题

1. 在github或者hexo-theme下载你喜欢的主题，将它添加到
  \blog\themes

1. 配置\blog\ _config.yml

    比如theme为next

        # Extensions
        ## Plugins: https://hexo.io/plugins/
        ## Themes: https://hexo.io/themes/
        theme: next

#### next添加分类页、标签页、个人页

[简书-hexo添加分类和标签](https://www.jianshu.com/p/e17711e44e00)

例如标签：

首先配置主题\blog\theme\ _config.yml

        menu:
        home: / || fa fa-home
        tags: /tags/ || fa fa-tags
        categories: /categories/ || fa fa-th
        archives: /archives/ || fa fa-archive
        about: /about/ || fa fa-user
        #schedule: /schedule/ || fa fa-calendar
        #sitemap: /sitemap.xml || fa fa-sitemap
        #commonweal: /404/ || fa fa-heartbeat

1. 在blog路径下使用命令

        hexo new page tags

2. 创建成功命令行会生成路径 blog/source/tags/index.md

    为其添加属性 type: "tags

        ---
        title: tags
        date: 2017-05-27 13:47:40
        type: "tags"
        ---

3. 给文章添加tags属性，例如

        ---
        title: STM32使用HAL库编写IIC驱动
        date: 2017-05-26 12:12:57
        categories:
        - STM32
        tags:
        - STM32
        - IIC
        ---
