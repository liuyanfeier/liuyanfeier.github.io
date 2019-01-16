### 工具安装

- 安装nodejs：http://nodejs.cn/download/
- sudo npm install hexo-cli -g
- sudo npm install -g --unsafe-perm hexo-cli

### 部署写作

```shell
- hexo new [layout] title       #新建文章，source/\_posts 目录下生成一个 .md 文件，可以在命令中指定文章的布局（layout），默认为 post。可以将 layout 设为 draft 那该文章就会被放入草稿箱 source/\_drafts。
- hexo publish [layout] title   #当文章被放入草稿箱 source/\_drafts 之后可以使用该命令将 source/\_drafts 文章放入 source/\_posts 中。
- hexo g                        #生成静态页面
- hexo d                        #部署到github
- hexo clean                    #清空所有生成的网页缓存
```

### git提交

在该项目中，master分支存放静态页面，hexo分支存放的是原始文件。当clone下整个项目后，新建文章生成静态页面部署等操作都在hexo分支上进行，hexo d部署成功之后master分支会自动更新，新的文章也会在网页上出现。
1. git add *
2. git commit -a -m "add new article"
3. git push origin hexo

### 写作技巧

1. 文章头部信息
    ```shell
    title:      viterbi以及forward-backword算法
    author:     liuyan
    catalog:    true
    tags:
        - 语音识别
        - viterbi
    date:       2019-01-07 17:05:01
    urlname:
    categories: 语音识别
    ```

2. 设置阅读全文：在文章中使用 `<!-- more -->` 手动进行截断 。

3. hexo插入图片方法：
    - 把\_config.yml里的post_asset_folder设为true
    - 在hexo目录下执行: `npm install hexo-asset-image –save`，下载安装一个可以上传本地图片的插件 
    - 运行hexo new “xxxx”来生成md博文时，source/\_posts文件夹内除了xxxx.md文件还有一个同名的文件夹 
    - 在xxxx.md中引入图片时，先把图片复制到xxxx这个文件夹中，然后只需要在xxxx.md中按照markdown的格式引入图片. `![](图片名.jpg)`

### 安装插件

```shell
npm install hexo-deployer-git --save
npm install hexo-asset-image --save
npm install hexo-math --save
npm install hexo-wordcount --save
npm install hexo-all-minifier --save
```

