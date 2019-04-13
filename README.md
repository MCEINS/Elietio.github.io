# [https://www.elietio.xyz](https://www.elietio.xyz "https://www.elietio.xyz")

## Setup

### Requirements
Installing Hexo is quite easy. However, you do need to have a couple of other things installed first:

- Node.js (Should be at least nodejs 6.9)
- Git
 
If your computer already has these, congratulations! Just install Hexo with npm:

	$ npm install -g hexo-cli

Once Hexo is installed, run the following commands to initialise Hexo in the target <folder>.

	$ hexo init <folder>
	$ cd <folder>
	$ npm install
Once initialised, here’s what your project folder will look like:

	.
	├── _config.yml
	├── package.json
	├── scaffolds
	├── source
	|   ├── _drafts
	|   └── _posts
	└── themes



> _config.yml

Site [configuration](https://hexo.io/zh-cn/docs/configuration.html "https://hexo.io/zh-cn/docs/configuration.html") file. You can configure most settings here.



> themes

[Theme](https://hexo.io/zh-cn/docs/themes "https://hexo.io/zh-cn/docs/themes") folder. Hexo generates a static website by combining the site contents with the theme.

Recommend [hexo-theme-next](https://github.com/theme-next/hexo-theme-next "https://github.com/theme-next/hexo-theme-next")

Related [documentation](http://theme-next.iissnan.com/ "http://theme-next.iissnan.com/")

### Generating and Deployment

With the release of Hexo 3, the server has been separated from the main module. To start using the server, you will first have to install hexo-server.
	
	$ npm install hexo-server --save
	$ hexo server or hexo s
	$ hexo generate or hexo g
	$ hexo deploy or hexo d


## Dependent plugins

> [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git "https://github.com/hexojs/hexo-deployer-git")

	$ npm install hexo-deployer-git –save

>  [hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed "https://github.com/hexojs/hexo-generator-feed")

    $ npm install hexo-generator-feed --save
> [hexo-generator-searchdb](https://github.com/theme-next/hexo-generator-searchdb "https://github.com/theme-next/hexo-generator-searchdb")

	$ npm install hexo-generator-searchdb --save

> [hexo-generator-sitemap](https://github.com/hexojs/hexo-generator-sitemap "https://github.com/hexojs/hexo-generator-sitemap")
	
	$ npm install hexo-generator-sitemap --save


> [hexo-generator-baidu-sitemap](https://github.com/coneycode/hexo-generator-baidu-sitemap "https://github.com/coneycode/hexo-generator-baidu-sitemap")

	npm install hexo-generator-baidu-sitemap --save

> [hexo-douban](https://github.com/mythsman/hexo-douban "https://github.com/mythsman/hexo-douban")

	$ npm install hexo-douban --save

> [hexo-helper-live2d](https://github.com/EYHN/hexo-helper-live2d "https://github.com/EYHN/hexo-helper-live2d")

	$ npm install hexo-helper-live2d --save



## others

	$ npm config set registry http://registry.npm.taobao.org/
	$ npm config set registry https://registry.npmjs.org/