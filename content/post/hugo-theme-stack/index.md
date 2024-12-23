---
title: "轻松搭建个人博客: 使用Hugo和Hugo Theme Stack完整指南"
description: "探索如何使用Hugo和Hugo Theme Stack主题,轻松创建一个个性化的博客,记录生活与思考"
date: 2024-12-07
image: 
draft: false
tags: []
categories: ["知识分享"]
---

作为一名程序开发者，拥有一个属于自己的博客已经不再是可有可无的事，而是提升个人品牌、记录技术成长的重要工具。之前，我尝试使用 Docsify 搭建了一个记录知识的博客，虽然它能够快速展示内容并支持动态加载 Markdown 文件，但在使用过程中我发现它更像是一个文档系统，而不是一个完整的博客平台。Docsify 提供的主题较为单一，灵活性也有限，尤其是在我想要根据不同需求切换主题、呈现不同视觉风格时，显得有些力不从心。

因此，我希望能够找到一个工具，既能保证内容的稳定性，又能轻松更换多个主题，享受不一样的视觉体验。这对于我来说非常重要，因为博客不仅是展示技术内容的工具，还是我与读者互动的桥梁，视觉上的多样性和体验感直接影响读者的参与感和浏览体验。最终我选择了Hugo作为搭建博客的工具。

## Hugo
 Hugo是什么？ 请点击这里🫱 [https://hugo.opendocs.io/about/what-is-hugo/](https://hugo.opendocs.io/about/what-is-hugo/)

### 安装Hugo
请点击这里🫱 [https://hugo.opendocs.io/installation/](https://hugo.opendocs.io/installation/)

* 安装之前，需要安装go、git
* Hugo不要安装标准版，请安装extend版本，因为支持Sass与Webp格式图片

### 创建站点

参考[https://hugo.opendocs.io/getting-started/quick-start/](https://hugo.opendocs.io/getting-started/quick-start/)
```yaml
 hugo new site blogs
 cd blogs
 hugo server  -D // 启动服务，包含草稿内容
```

### 使用Hugo Theme Stack主题
官方文档 [https://stack.jimmycai.com/guide/](https://stack.jimmycai.com/guide/)

官方Demo网站 [https://demo.stack.jimmycai.com/](https://demo.stack.jimmycai.com/)

Hugo Theme Stack 是一个非常受欢迎的主题，支持搜索、标签、分类、归档、暗黑模式、多语言等多种功能，适合用来搭建现代化的博客。下面是如何将 Hugo Theme Stack 集成到你的博客中的步骤：
#### 下载主题
首先，在项目的根目录下，执行以下命令以将 Hugo Theme Stack 下载到 themes/ 目录中：
```yaml
git clone https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack
```
也可以通过git submodule的方式
```yaml
git submodule add https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack
```
#### 更新配置文件
接下来，你需要更新一些配置文件。具体步骤如下：
* 删除站点根目录中的原有 hugo.toml 文件。
* 将 exampleSite 文件夹中的 hugo.yaml 复制到站点的根目录。
* 将 exampleSite/content 目录中的内容替换为你的站点根目录中的 content 文件夹。
#### 启动 Hugo 服务
完成以上步骤后，重启 Hugo 服务，执行以下命令来查看效果：
```yaml
hugo server -D
```
你将能够看到 Hugo Theme Stack 主题已经生效。此时，你的博客已经搭建完成，新的主题已经应用
#### 切换主题
未来如果你需要更换主题，只需按照相同的方式下载新的主题（例如使用 git clone 命令），然后修改配置文件中的 theme 字段，将其指向你下载的新主题。例如：
```yaml
theme: 你的主题名称
```
## 发布博客

现在，我们已经完成了 Hugo Theme Stack 主题的集成，接下来是将博客发布到线上。可以选择将博客托管在 GitHub Pages、Netlify 或其他静态网站托管平台上。具体的发布方式取决于你选择的托管平台。

### 将Hugo部署到Github Pages

参考 [https://hugo.opendocs.io/hosting-and-deployment/hosting-on-github/](https://hugo.opendocs.io/hosting-and-deployment/hosting-on-github/)。

Github Pages一个账号只能有一个免费的使用名额。

### 使用Devbox部署Hugo

Devbox是一个一站式博客开发与部署的平台。

优点：不用部署Hugo环境、开箱即用、支持ssh开发、支持自定义域名。

缺点：收费，自定义域名有一定的学习成本，需要购买域名与服务器，备案域名后才能通过CNAME方式自定义域名。

为了数据安全，有必要将代码推送到github。

参考 [https://mp.weixin.qq.com/s/F3orZXjk6wQ23BJRj3dsvw](https://mp.weixin.qq.com/s/F3orZXjk6wQ23BJRj3dsvw)