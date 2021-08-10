技术栈：
gitbook + markdown + git

本地生成电子书：
1. npm install -g gitbook-cli 全局安装
2. gitbook init	在本地创建一个新电子书项目
3. 编辑目录+内容
4. gitbook serve	本地运行，本地效果跟线上效果是有出入的
注：如果已通过web app创建的电子书并且链接了git仓库，则可直接拉取git然后执行第4步

使用插件：
1. 根目录下新建book.json(等同于package.json)
2. book.json内容如下所示：
```json
{
	"title": "网络日志",
	"author": "LY",
	"description": "",
	"language": "zh-hans",
	"plugins": [
		"back-to-top-button",
		"hide-element"
	],
	"pluginsConfig": {
		"hide-element": {
			"elements": [
				".gitbook-link"
			]
		}
	}
}

```
[官网上提供的部分插件](https://docs.gitbook.com/resources/gitbook-legacy/v2-differences#plugins)
3. gitbook install	安装依赖
4. gitbook serve	本地运行查看效果，本地效果跟线上效果是有出入的