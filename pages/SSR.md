##### 客户端渲染（CSR） vs 服务端渲染（SSR）

**1. 什么是CSR**

`客户端渲染` 即服务器把数据传给浏览器，浏览器根据静态文件吧数据渲染到页面上展示。

当前流行框架制作的SPA应用不利于SEO，因为SPA应用有很多路由但是页面只有一个，所有页面都是js即时生成的dom。

**2. 什么是SSR**

`服务端渲染` 即在服务器端渲染好页面然后传给浏览器展示。

能够解决SEO问题，使其实现所有应用路径能够被收录。

**3. CSR 与 SSR 优缺点对比**

## Nuxt.js

>  [中文官网][https://www.nuxtjs.cn/guide]

### 目录结构

```bash
components -Vue.js组件，无`asyncData`方法
layouts -布局，通用模版，有`asyncData`方法
middleware -中间件，定义一个自定义函数运行在一个页面或一组页面渲染之前
pages -页面视图
plugins -第三方插件、库
server - 服务端应用
assets -需要被`Webpack`处理的静态资源
static -不需要被`Webpack`处理的静态资源，访问时不用写`static/`
store -vuex
test -测试

.editorconfig -配置代码规范
nuxt.config.js -配置Nuxt.js
package.json -描述应用的依赖关系（大版本号）和对外暴露的脚本接口，拉取的依赖是该大版本下的最新版本
package-lock.json -描述安装依赖的具体版本号，最终拉取依赖是用这个文件描述的版本号
```

| 别名         | 目录                                                       |
| ------------ | ---------------------------------------------------------- |
| `~` 或 `@`   | [srcDir](https://www.nuxtjs.cn/api/configuration-srcdir)   |
| `~~` 或 `@@` | [rootDir](https://www.nuxtjs.cn/api/configuration-rootdir) |

默认情况下，`srcDir` 和 `rootDir` 相同，都指向根目录。引入 `assets` 或者 `static` 目录时需要使用别名。

### 生命周期

红色：仅运行在服务端

黄色：运行在服务端、客户端

绿色：vue的生命周期，运行在客户端

![nuxt生命周期](/.gitbook/assets/nuxt生命周期.png)

执行顺序：

1. nuxtServerInit: 初始化服务器
2. middleware: 中间件
   1. nuxt.config.js
   2. 匹配布局
   3. 匹配页面
3. validate: 参数校验
4. asyncData: 异步获取数据返给组件
5. fetch: 异步获取数据返给vuex
6. beforeCreate: 创建前，服务端、客户端各执行一次
7. created: 创建后，服务端、客户端各执行一次
8. vue其他生命周期（不按执行顺序排）
   1. beforeMount
   2. mounted
   3. beforeUpdate
   4. updated
   5. activated
   6. deactivated
   7. beforeUnmount
   8. unmounted
   9. errorCaptured
   10. renderTracked
   11. renderTriggered

### 视图

Nuxt.js应用生成视图的过程：

1. 匹配模板：src 文件夹下（默认是应用根目录）的 `app.html`。
2. 匹配布局：`layouts/default.vue` 默认为全局布局无需在页面中声明， `error.vue` 是默认的错误页面，自定义的布局文件需要声明。
3. 匹配页面：`pages/index.vue` 默认为应用首页，所有页面跟spa应用页面的区别仅仅是增加了 Nuxt.js 提供的功能特性（配置项）。

![Nuxt.js应用生成视图的过程](https://www.nuxtjs.cn/nuxt-views-schema.svg)

### 路由

**1. 路由配置**

Nuxt.js 默认采取的 `约定式路由` 会依据 `pages` 目录结构自动生成 [vue-router](https://github.com/vuejs/vue-router) 模块的路由配置。

- 基础路由
- 动态路由

[动态路由](https://www.nuxtjs.cn/guide/routing#%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1)

- 动态路由参数校验

在动态路由组件中定义参数校验方法：

```vue
// pages/users/_id.vue
export default {
  validate({ params }) {
    // 必须是number类型，返回 false 则自动加载错误页面
    return /^\d+$/.test(params.id)
  }
}
```

- 嵌套路由

[嵌套路由](https://www.nuxtjs.cn/guide/routing#%E5%B5%8C%E5%A5%97%E8%B7%AF%E7%94%B1)

- 扩展路由-声明式路由配置

要使用 `声明式路由`，需要在 `nuxt.config.js` 文件中扩展路由器配置：

```js
// 需要配置项：routes
// 路径：/index
// 匹配页面：index.vue
export default {
  router: {
    extendRoutes(routes, resolve) {
      console.log(routes)
      routes.push({
          name: 'home',
          path: 'index',
          component: resolve(__dirname, 'pages/index.vue')
      })
    }
  }
}
```

**2. 路由跳转**

路由跳转采取 [声明式](https://next.router.vuejs.org/zh/guide/essentials/navigation.html#%E5%AF%BC%E8%88%AA%E5%88%B0%E4%B8%8D%E5%90%8C%E7%9A%84%E4%BD%8D%E7%BD%AE)，详情参看 [router-link](https://router.vuejs.org/zh-cn/api/router-link.html)。

```html
<nuxt-link to="/about">关于</nuxt-link>
```

**3. 显示页面内容**

- 布局中显示页面内容

```vue
<nuxt />

// 命名视图
<nuxt name="top" />

// 传递props
<nuxt :nuxt-child-key="someKey" />
```

- 嵌套路由显示页面内容

```vue
// 父页面
<nuxt-child :foobar="123" />

// 使用 keep-alive 组件
<nuxt-child keep-alive :keep-alive-props="{ exclude: ['modal'] }" />

// 命名视图
<nuxt-child name="top" />
```

- 使用命名视图显示页面内容

要指定页面的 `命名视图`，需要在 `nuxt.config.js` 文件中扩展路由器配置：

```js
// 需要配置项：components、chunkNames
// 命名视图名称为：top
export default {
  router: {
    extendRoutes(routes, resolve) {
      const index = routes.findIndex(route => route.name === 'main')
      routes[index] = {
        ...routes[index],
        components: {
          default: routes[index].component,
          top: resolve(__dirname, 'components/mainTop.vue')
        },
        chunkNames: {
          top: 'components/mainTop'
        }
      }
    }
  }
}
```

**4. 导航守卫**

- 前置守卫

在 `nuxt.config.js` 文件中扩展路由器配置：

```js
export default {
  router: {
    middleware: ['authenticated']
  }
}
```

在 `middleware/authenticated.vue` 中配置全局前置守卫：

```vue
export default function ({ store, redirect }) {
  console.log('middleware: ', '配置级中间件')
  // 全局前置导航守卫
  console.log('触发全局前置导航守卫');
  // 重定向到 /pages/three.vue
  return redirect('/three')
}
```

在 `layout/default.vue` 或 `组内` 中可配置布局级和组件内的前置守卫：

```vue
export default {
  middleware({ redirect }) {
    return redirect('/three')
  }
}
```

- 后置守卫

通过 `beforeRouteLeave` 设置组件内的后置守卫：

```vue
export default {
  beforeRouteLeave(to, from, next) {
    let foo = window.confirm('是否要离开')
    next(foo)
  }
}
```

- 通过 plugins 使用 vue-router 导航守卫

`plugins` 中能够自由使用客户端钩子。

在 `plugins/router.js` 中：

```js
export default ({ app, redirect }) => {
  // app: vue实例  redirect: 重定向
  // 前置钩子 beforeEach
  app.router.beforeEach((to, from, next) => {
    let arr = ['index', 'one', 'two']
    if (arr.includes(to.name)) {
      next()
    } else {
      redirect({ name: 'two'} )
      next()
    }
  })
}
```

在 `nuxt.config.js` 文件中扩展插件配置：

```js
export default {
  plugins: ['@/plugins/router']
}
```

### 过渡动效

Nuxt.js 默认使用的过渡效果名称为 `page`，Vue.js 默认使用的是 `v`。

过渡效果类名参看：[过渡class](https://v3.cn.vuejs.org/guide/transitions-enterleave.html#%E8%BF%87%E6%B8%A1class)。

**1. 全局页面过渡效果**

全局样式文件 `assets/main.css`：

```css
/* 动画效果淡入、淡出 */ ？类名有误
// 设置全局过渡效果时 page 是默认名称
.page-enter-active,
.page-leave-active {
  transition: opacity .5s;
}
.page-enter,
.page-leave-active {
  opacity: 0;
}
```

在 `nuxt.config.js` 中扩展css配置：

```js
module.exports = {
  css: ['@/assets/main.css']
}
```

全局过渡效果无需在页面中使用 `transition` 属性声明过渡效果名称。

**2. 定制页面过渡效果**

全局样式文件 `assets/main.css` 或 页面style：

```css
/* 动画效果移入、移出 */
// 自定义过渡效果名称 test
.test-enter-active,
.test-leave-active {
  transition: .5s ease all;
}
.test-enter,
.test-leave-active {
  margin-left: -1000px;
}
```

页面中声明过渡效果名称：

```vue
export default {
  transition: 'test'
}
```

**3. 数据加载loading效果**

`nuxt.config.js` 中配置系统默认的 loading 效果或指定一个 loading 组件：

```js
export default {
  // 二选一
  loading: { color: '#399', height: '5px' } // 配置系统默认loading样式
  loading: '@/components/loading.vue' // 指定自定义loading组件
}
```

创建 `components/loading.vue` 文件：

```vue
<template>
  <div v-if="loading">
    loading...
  </div>
</template>
<script>
  export default {
    data() {
      return {
        loading: false
      }
    },
    methods: {
      // nuxt.js loading内置方法
      start() {
        this.loading = true
      },
      // nuxt.js loading内置方法
      finish() {
        this.loading = false
      }
    },
  }
</script>
```

### axios数据交互

- asyncData: 在组件初始化前被调用，因为没有实例组件所以无法使用this，异步获取到的数据通过 return 返给组件使用。

- fetch: 类似 asyncData，但它是把数据返给 Vuex。

**1. 获取同域资源**

在 `static` 目录下创建 `title.json`作为同域资源。（静态伺服）

组件内：

```vue
export default {
  async asyncData({ $axios }) {
    let res = await $axios({ url: '/title.json' })
    console.log("读取到的静态资源: ", res.data);
    return {
      listData: res.data // 组件内使用listData
    }
  }
}
```

**2. 获取跨域资源**

> 协议、域名、端口有一个不同则为跨域。

组件内：

```vue
export default {
  asyncData({ $axios, error }) {
    return $axios({ url: 'http://localhost:3001/api/homenav' }).then(res => {
      console.log(res.data);
      return res.data
    }).catch(err => {
      // 加载错误布局
      error({
        statusCode: 500,
        message: '跨域'
      })
    })
  }
}
```

在 `nuxt.config.js` 中配置 `proxy`：

```js
export default {
  modules: ['@nuxtjs/axios'],
  axios: {
    proxy: true // 开启axios跨域
  }，
  proxy: {
    '/api/': {
      target: 'http://localhost:3001', // 代理转发的地址
      changeOrigin: true,
      pathRewrite: {
        '^/api/': '/' // 重写路径
      }
    }
  }
}
```

**3. 拦截器**

创建 `plugins/axios.js` 文件：

```js
export default ({ $axios, redirect, error }) => {
  // 基本配置
  $axios.defaults.timeout = 10000
  
  // 请求拦截
  $axios.onRequest(config => {
    console.log('请求拦截');
    config.headers.token = ''
    return config
  })
  
  // 响应拦截
  $axios.onResponse(res => {
    console.log("响应拦截: ",res);
    if (res.status !== 200) {
      console.log("响应拦截处打印");
      redirect('/index') // 重定向到index.vue
    }
  })
  
  // 错误处理
  $axios.onError(error => {
    console.log("错误处理处打印");
    error({ // 跳转至错误布局
      statusCode: 500,
      message: error
    })
  })
}
```

在 `nuxt.config.js` 中配置 `plugins`：

```js
export default {
  plugins: [
    '@/plugins/axios'
  ]
}
```

### Vuex状态树

### 资源配置

### TypeScript

