# 使用手册

## 如何从零开始搭建 Hexo 博客

1. 参考 [Hexo 安装](https://hexo.io/zh-cn/docs/)， 添加 Hexo，接着初始化项目，且命名为 `book`：

```sh
$ hexo init book
```

2. 安装一键部署工具：

```sh
$ cd book
$ npm install https://github.com/xinetzone/hexo-deployer-git.git
```

3. 修改网站配置 `book/_config.yml`，以匹配所需。

4. 生成静态网页，并部署到 GitHub Pages：

```sh
$ hexo clean && hexo g
$ hexo d
```

## 直接使用 `notebook` 作为你的博客模板

1. 进入 [xinetzone/notebook](https://github.com/xinetzone/notebook)（修改自 [blinkfox/hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery)） 选择按钮 `Use this template` 即可直接使用。
2. 当然，仍需要修改 `book/_config.yml` 和 `book/_config.matery`，以匹配所需。
3. 将此仓库克隆到本地，进入 `book/themes`，添加所需要的主题插件（你也可以替换为其他主题插件）：

```sh
$ git clone https://github.com/xinetzone/matery.git
```
4. 这样便可以开箱即用。
5. 修改 `book/themes/matery/layout/_partial/social-link.ejs` 可以添加新的联系方式。
6. 添加新的目录或者子目录，可以修改 `book/themes/matery/layout/_partial/navigation.ejs` 和 `book/_config.matery.yml` 的 `menu`。、