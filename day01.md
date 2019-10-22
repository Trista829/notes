# 项目内容

├── build                      // 构建相关  
├── config                     // 配置相关
├── src                        // 源代码
│   ├── api                    // 所有请求
│   ├── assets                 // 主题 字体等静态资源
│   ├── components             // 全局公用组件
│   ├── directive              // 全局指令
│   ├── filtres                // 全局 filter
│   ├── icons                  // 项目所有 svg icons
│   ├── lang                   // 国际化 language
│   ├── mock                   // 项目mock 模拟数据
│   ├── router                 // 路由
│   ├── store                  // 全局 store管理
│   ├── styles                 // 全局样式
│   ├── utils                  // 全局公用方法
│   ├── vendor                 // 公用vendor
│   ├── views                   // view
│   ├── App.vue                // 入口页面
│   ├── main.js                // 入口 加载组件 初始化等
│   └── permission.js          // 权限管理
├── static                     // 第三方不打包资源
│   └── Tinymce                // 富文本
├── .babelrc                   // babel-loader 配置
├── eslintrc.js                // eslint 配置项
├── .gitignore                 // git 忽略项
├── favicon.ico                // favicon图标
├── index.html                 // html模板
└── package.json               // package.json

## 1、views

根据学习模块来划分views，并且将views和api两个模块一一对应

## 2、components

放置全局的组件，一些页面级组件建议放在各自的views文件下

## 3、ESLint

在setting.config中添加了如下配置

    "files.autoSave":"off",
    "eslint.validate": [
       "javascript",
       "javascriptreact",
       "html",
       { "language": "vue", "autoFix": true }
     ],
     "eslint.options": {
        "plugins": ["html"]
     }

## 4、axios

封装axios，在本地调试线上bug

## 5、多环境配置

添加   sit测试环境  和   stage预发布环境，如下

```json
"build:prod": "NODE_ENV=production node build/build.js",
"build:sit": "NODE_ENV=sit node build/build.js",
```

在新版vue-cli中，webpack-bundle-analyzer用来模块分析，如下

```json
//package.json
"build:sit-preview": "cross-env NODE_ENV=production env_config=sit npm_config_preview=true  npm_config_report=true node build/build.js"
//之后通过process.env.npm_config_report来判断是否来启用webpack-bundle-analyzer
var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
webpackConfig.plugins.push(new BundleAnalyzerPlugin())
```

## 6、跨域问题

cors
或 webpack-dev-server的proxy解决，开发环境用nginx反代理解决

## 7、模拟数据

mockjs

## 8、图标组件

```vue
//components/Icon-svg
<template>
  <svg class="svg-icon" aria-hidden="true">
    <use :xlink:href="iconName"></use>
  </svg>
</template>

<script>
export default {
  name: 'icon-svg',
  props: {
    iconClass: {
      type: String,
      required: true
    }
  },
  computed: {
    iconName() {
      return `#icon-${this.iconClass}`
    }
  }
}
</script>

<style>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```

```javascript
//引入svg组件
import IconSvg from '@/components/IconSvg'
//全局注册icon-svg
Vue.component('icon-svg', IconSvg)
//在代码中使用
<icon-svg icon-class="password" />
```

## 9、svg-script

动态生成icont字体图标，不用每次整体删除重新加载包

使用 `svg-sprite-loader`，是一个webpack loader，可以将多个svg打包成svg-sprite

在vue-cli的基础上进行改造，加入svg-sprite-loader，但vue-cli中默认使用了url-loader对svg进行处理，并放在/img目录下。所以此时引入svg-sprite-loader会引发一些冲突

```javascript
//默认`vue-cli` 对svg做的处理，正则匹配后缀名为.svg的文件，匹配成功之后使用 url-loader 进行处理。
 {
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    loader: 'url-loader',
    options: {
      limit: 10000,
      name: utils.assetsPath('img/[name].[hash:7].[ext]')
    }
}

```

解决方法：

使用webpack的 exclude 和 include ，让svg-sprite-loader只处理指定文件夹下的svg，url-loader只处理除此文件夹之外的所有svg

```javascript
{
    test:/\.svg$/,
    loader:'svg-sprite-loader',
    include:[resolve('src/icons')],
    options:{
        symbolId:'icon-[name]'
    }
},
{
    test:/\.(png|jpe?g|gif|svg)(\?.*)?$/,
    loader:'url-loader',
    include:[resolve('src/icons')],
    options:{
        limit:10000,
        name:utils.assetsPath('img/[name].[hash:7].[ext]')
    }
}
```

使用：

```vue
import '@/src/icons/qq.svg; //引入图标

<svg><use xlink:href="#qq" /></svg>  //使用图标
```

   

# element UI

## 安装

1、通过npm安装

`npm i element-ui --save`

2、通过CDN获取最新资源

```html
<!-- 引入样式 -->
<link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
<!-- 引入组件库 -->
<script src="https://unpkg.com/element-ui/lib/index.js"></script>
```

## 引入使用

在main.js文件中添加如下代码

### 1、完整引入

```javascript
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

//全局配置：拥有size属性的组件默认尺寸small，弹框的初始zindex为3000
Vue.use(ElementUI,{size:'small',zIndex:3000})
```

### 2、按需引入

借助 babel-plugin-component，可以按需引入，减少项目体积

①安装babel-plugin-component

`npm install babel-plugin-component -D`

②将.babelrc修改为

```javascript
{
    "presets":[["es2015",{"modules":false}]],
    "plugins":[
        [
            "component",
            {
                "libraryName":"element-ui",
                "styleLibraryName":"theme-chalk"
            }
        ]
    ]
}
```

③引入部分组件，例如Button 和 Select ，在main.js中如下

```javascript
import { Button, Select } from 'element-ui'

Vue.component(Button.name, Button)
Vue.component(Select.name, Select)
/*
或者
Vue.use(Button)
Vue.use(Select)
*/

//全局配置
Vue.prototype.$ELEMENT = {size:'small',zIndex:3000};
```

## 自定义主题

1、在线主题编辑器 

https://element.eleme.io/#/zh-CN/theme 

2、在线主题生成工具

https://elementui.github.io/theme-chalk-preview/#/zh-CN 

3、在项目中改变scss变量

```scss
/* 改变主题色变量 */
$--color-primary: teal;

/* 改变 icon 字体路径变量，必需 */
$--font-path: '~element-ui/lib/theme-chalk/fonts';

@import "~element-ui/packages/theme-chalk/src/index";
```

```javascript
import Vue from 'vue'
import ElementUI from 'element-ui'

//注意此处的添加修改！！！
import './element-variables.scss'

Vue.use(ElementUI)
```

4、命令行工具

①安装主题生成工具

`npm i element-theme `

②安装主题

`npm i element-theme-chalk -D`

③初始化变量文件

` et -i [可以自定义变量文件] > ` 

`✔ Generator variables file `

④修改变量

直接编辑 element-variaables.scss文件

$--color-primary:red;

⑤编译主题

`et`

# vue-cli

## 创建步骤

### 1、安装

`npm install -g webpack`

`npm install -g @vue/cli`

### 2、即时原型制作（可无吧）

`npm install -g @vue/cli-service-global`

### 3、初始化

cmd命令中通过cd进入要放置项目的文件夹。在命令窗口  `vue init webpack 文件名`  然后一系列回车创建一个初始化项目

### 4、进入刚刚创建的项目文件夹

### 5、启动项目

`npm run dev`

## 目录列表

```bash
├── build/                      # webpack 编译任务配置文件: 开发环境与生产环境
│   └── ...
├── config/                     
│   ├── index.js                # 项目核心配置
│   └── ...
├ ── node_module/               #项目中安装的依赖模块
   ── src/
│   ├── main.js                 # 程序入口文件
│   ├── App.vue                 # 程序入口vue组件
│   ├── components/             # 组件
│   │   └── ...
│   └── assets/                 # 资源文件夹，一般放一些静态资源文件
│       └── ...
├── static/                     # 纯静态资源 (直接拷贝到dist/static/里面)
├── test/
│   └── unit/                   # 单元测试
│   │   ├── specs/              # 测试规范
│   │   ├── index.js            # 测试入口文件
│   │   └── karma.conf.js       # 测试运行配置文件
│   └── e2e/                    # 端到端测试
│   │   ├── specs/              # 测试规范
│   │   ├── custom-assertions/  # 端到端测试自定义断言
│   │   ├── runner.js           # 运行测试的脚本
│   │   └── nightwatch.conf.js  # 运行测试的配置文件
├── .babelrc                    # babel 配置文件
├── .editorconfig               # 编辑配置文件
├── .gitignore                  # 用来过滤一些版本控制的文件，比如node_modules文件夹 
├── index.html                  # index.html 入口模板文件
└── package.json                # 项目文件，记载着一些命令和依赖还有简要的项目描述信息 
└── README.md                   #介绍自己这个项目的，可参照github上star多的项目。
build/
```

