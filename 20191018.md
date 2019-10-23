# vue-router

## 创建

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <router-view></router-view>
</div>
```

```javascript
//定义路由组件
const Foo = {template:'<div>foo</div>'}
const Bar = {template:'<div>bar</div>'}

const router = new VueRouter({
    routes:[
        {path:'/foo',component:Foo},
        {path:'/bar',component:Bar}
    ]
})

const app = new Vue({
    router:router
    //router
}).$mount('#app')
```

通过注入路由器，可以访问他  `this.$router`  以及  `this.$route`  任何组件内部的当前路由：

```javascript
//Home.vue
export default {
	computed: {
        username(){
            return this.$route.params.username
        }
    },
    methods:{
        goBack(){
            window.history.length > 1 ? this.$router.go(-1) : this.$router.push('/')
        }
    }
}
```

# 项目

## 整体布局

![1571366297109](C:\Users\liujinting_sx\AppData\Roaming\Typora\typora-user-images\1571366297109.png)

## 路由

constantRoutes：不需要动态判断权限的路由，如登录页，404等通用页面

asyncRoutes：需要动态判断权限并通过addRoutes动态添加的页面



（在参考项目中都是用了路由懒加载，如下）

```javascript
const Foo = () => import('./Foo.vue')
```



## 侧边栏（Sidebar）

侧边栏是通过读取路由来动态生成的，且支持路由无限嵌套

原则上是有多少级路由嵌套，就需要多少个<router-view>

```javascript
//没有子路由
{
    path:'/icon',
    component: Layout,
    children: [{
        path:'index',
        component:()=>import('svg-icons/index'),
        name:'icons',
        meta:{title:'icons',icon:'icon'}
    }]
}

//有子路由
{
    path:'/components',
    component: Layout,
    name:'component-demo',
    meta:{
        title:'components',
        icon:'component'
    },
    children:[
        {
            path:'tinymce',
            component:()=>import('components-demo/tinymce'), 
            name: 'tinymce-demo', 
            meta: { title: 'tinymce'}
        },
        { 
        	path: 'markdown', 
        	component: ()=>import('components-demo/markdown'), 
            name: 'markdown-demo', 
            meta: { title: 'markdown' }
		}
    ]
}
```

**点击侧边栏 刷新当前路由**

方法一：

```javascript
clickLink(path) {
  this.$router.push({
    path,
    query: {
      t: +new Date() //保证每次点击路由的query项都是不一样的，确保会重新刷新view
    }
  })
}
```

 ps:

不要忘了在 `router-view` 加上一个特定唯一的 `key`，如 ,`<router-view :key="$route.path"></router-view>`但这也有一个弊端就是 url 后面有一个很难看的 `query` 后缀如 `xxx.com/article/list?t=1496832345025` 

方法二：

判断当前点击菜单的路由和当前的路由是否一致，如果一致，则跳转到一个专门redirect的页面，然后将路由重定向到我想去的页面，起到了刷新的效果

点击的时候重定向页面至`/redirect`

```javascript
const { fullpath } = this.$route
this.$router.replace({
    path:'/redirect'+fullpath
})
```

`redirect`页面重定向回原始页面

```javascript
//redirect.vue
export default {
    beforeCreate(){
        const { params, query } = this.$route
        const { path } = params
        this.$router.replace({
            path:'/'+path,query
        })
    },
    render:function(h){
        return h()
    }
}
```

## 面包屑

 https://github.com/PanJiaChen/vue-element-admin/blob/master/src/components/Breadcrumb/index.vue 

## 侧边栏滚动  el-scrollbar

注意：需要手动触发更新！！！

```html
<el-scrollbar wrap-class="scrollbar-wrapper">
      <el-menu
        :default-active="activeMenu"
        :collapse="isCollapse"
        :background-color="variables.menuBg"
        :text-color="variables.menuText"
        :unique-opened="false"
        :active-text-color="variables.menuActiveText"
        :collapse-transition="false"
        mode="vertical"
      >
        <sidebar-item v-for="route in routes" :key="route.path" :item="route" :base-path="route.path" />
      </el-menu>
</el-scrollbar>
```

 ![constructor](https://user-gold-cdn.xitu.io/2019/1/21/1686f8b54408c5e2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

|   参数    |                说明                |  类型   | 可选值 | 默认值 |
| :-------: | :--------------------------------: | :-----: | :----: | :----: |
| wrapClass |    可选参数，容器的样式名(外层)    | string  |   -    |   -    |
| viewClass | 可选参数，展示视图的样式名（内层） | string  |   -    |   -    |
| wrapStyle |        可选参数，容器的样式        | string  |   -    |   -    |
| viewStyle |      可选参数，展示视图的样式      | string  |   -    |   -    |
|  native   |     可选参数，是否使用原生滚动     | boolean |   -    | false  |
| noresize  |  可选参数，容器大小是否是不可变的  | boolean |   -    | false  |
|    tag    |      可选参数，渲染容器的标签      | string  |   -    |  div   |

<img src="C:\Users\liujinting_sx\AppData\Roaming\Typora\typora-user-images\1571378376227.png" alt="1571378376227"  />

## 固钉

面包屑下面那部分小标签

```javascript
 {
    path: '',
    component: Layout,
    redirect: 'dashboard',
    children: [
      {
        path: 'dashboard',
        component: () => import('@/views/dashboard/index'),
        name: 'Dashboard',
        meta: {
          title: 'dashboard',
          icon: 'dashboard',
          noCache: true,
          affix: true
        }
      }
    ]
  }
```

# 新增页面

1、在`@/router/index.js`中添加新路由

```javascript
//例如新增一个excel页面
{
    path: 'excel',
    component: Layout,
    redirect: '/excel/export-excel',
    name: 'excel',
    meta: {
      title: 'excel',
      icon: 'excel'
    },
    // 二级菜单
    children: [{
        path: 'export-excel',
        component: () => import('@/views/excel/exportExcel/index'),
        name: 'ExportExcel',
        meta: {
          title: 'ExportExcel'
        }
      },
      {
        path: 'upload-excel',
        component: () => import('@/views/excel/uploadExcel/index'),
        name: 'UploadExcel',
        meta: {
          title: 'UploadExcel'
        },
            
        //!!!!!!!!!!!!!!!!!!!!!!!
        // 三级菜单，添加三级路由的时候，别忘了在二级的index中添加<router-view>
        children:[
          {
            path:'two',
            component:()=>import('@/views/excel/uploadExcel/two'),
            name:'Two',
            meta:{
              title:'Two'
            }
          },
          {
            path:'third',
            component:()=>import('@/views/excel/uploadExcel/third/index'),
            name:'Third',
            meta:{
              title:'Third'
            }
          }
        ]
      }
    ]
  },

//同时应该在views目录下新建一个excel目录，里面添加一个新的组件文件夹，命名为exportExcel，文件夹中放index.vue
//若children下面有多个路由组件，则侧边箭头自动出现
      
？？问题？？
为什么此时uploadExcel和third下的index不写会报错找不到文件
```

![1571388230119](C:\Users\liujinting_sx\AppData\Roaming\Typora\typora-user-images\1571388230119.png)

![1571388253526](C:\Users\liujinting_sx\AppData\Roaming\Typora\typora-user-images\1571388253526.png)

## 新增view

新增路由后别忘了在views文件夹下创建对应的文件夹，一般一个路由对应一个文件，该模块下的功能组件或者方法就建议在本文件夹下创建一个components或utils文件夹，各个模块维护自己的components或utils

## 新增api

在@/api文件夹下创建本模块对应的api服务

## 新增组件

在全局的 `@/components` 只会写一些全局的组件，如富文本，各种搜索组件，封装的日期组件等等能被公用的组件。每个页面或者模块特定的业务组件则会写在当前 views 下面。如：`@/views/article/components/xxx.vue`。这样拆分大大减轻了维护成本。

**请记住拆分组件最大的好处不是公用而是可维护性！**

## 新增样式

```css
<style>

</style>

/*scoped 仅自己使用，避免造成全局样式污染*/
<style scoped>
.xxx-container{
  xxx
}
</style>
```

# 样式

## 覆盖element-ui样式

 现在我们来说说怎么覆盖 element-ui 样式。由于 element-ui 的样式我们是在全局引入的，所以你想在某个页面里面覆盖它的样式就不能加 scoped，但你又想只覆盖这个页面的 element 样式，你就可在它的父级加一个 class，用命名空间来解决问题。 

```css
.article-page {
  /* 你的命名空间 */
  .el-tag {
    /* element-ui 元素*/
    margin-right: 0px;
  }
}
```

## 父组件改变子组件样式 深度选择器

当你子组件使用了 `scoped` 但在父组件又想修改子组件的样式可以 通过 `>>>` 来实现：

```css
<style scoped>
.a >>> .b { /* ... */ }
</style>
```

将会编译成

```css
.a[data-v-f3f3eg9] .b {
  /* ... */
}
```

# 和服务端进行交互

。。。