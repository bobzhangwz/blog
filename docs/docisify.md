# 0 基础搭建博客

`docsify` 是一个文档网站生成工具，它可以将 markdown 文档发布到 github page 上，可以用来搭建个人博客，非常容易上手。

通过使用 `docsify`，我们可以熟悉一些简单的编程工具，如 git, github, markdown。下面让我们用 docsify 快速搭建一个博客。

## 准备工作

1. 申请一个github 帐号，并设置 SSH 密钥
  1. [生成密钥](https://docs.github.com/cn/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)
生成密钥时，可以不不设置密码，不要设置 ssh-agent
  2. [将生成的密钥公钥，添加到 github](https://docs.github.com/cn/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)
3. [安装git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
4. 安装 nodejs 运行环境, [安装手册](https://nodejs.org/zh-cn/download/package-manager/)
5. 学习 [markdown 语法](https://www.jianshu.com/p/q81RER)

## 初始化

1. 打开终端, 创建博客文档目录
```shell
# 假设你要再 ~/Documents/blog/ 下管理博客文件
mkdir -p ~/Documents/blog/
cd ~/Documents/blog/
```
2. 初始化项目，可以根据[这篇文档](https://docsify.js.org/#/zh-cn/quickstart)来初始化, 主要命令如下
```
npm i docsify-cli -g
docsify init .
```
3. 通过运行 `docsify serve .` 启动一个本地服务器，可以方便地实时预览效果。默认访问地址 http://localhost:3000

## 页面设置

1. 设置主页，在创建好的文件中 `README.md` 编辑，内容可以作为首页
2. [设置侧边](https://docsify.js.org/#/zh-cn/more-pages?id=%e5%ae%9a%e5%88%b6%e4%be%a7%e8%be%b9%e6%a0%8f)

## 发布

1. [创建github 仓库](https://docs.github.com/cn/github/getting-started-with-github/create-a-repo), 最好仓库名与你的用户名一致
2. 根据创建仓库后的提示，初始化项目，代码大致如下
```
git init .
git add .
git commit -m "first commit"
git push origin master
```
3. [设置github page](https://docs.github.com/cn/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

## 更新

如果有新的文章需要发布，可以执行
```
git add .
git commit -m "some message"
git push origin master
```
