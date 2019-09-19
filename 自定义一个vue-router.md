### 路由工作流程
1. URL发生改变
2. 触发监听事件，vue-router触发Window自带监听
3. 改变vuerouter里的current变量
4. 监视current私有变量的监听者被处罚，监视者设计模式，vue监视者
5. 获取新组件
6. render新组件
### Hash与History
######  Hash#后面的都是hash值，是location里自带的hash属性，可获取，并不是vue 的属性，没有#时候，location.hash拿到空字符串，可以通过 onhashchange监听hash的改变

###### History 正常浏览器路径，通过location.pathname来获取域名后面的部分?前面的部分，可以用onpopstate监听history改变

### vue插件
1. vue-router vuex 都是vue插件
2. Vue.use() 
##### 接收到一个function就执行这个function；
##### 如果接收到一个对象，就执行对象的install属性对应的function，如果没有install方法就直接调用这个对象
```
Vue.use() // 使用插件，将插件注册到vue实例里面
```
3. Vue.mixin 往vue全局（原型中）混入自定义操作

```
Vue.mixin({
    created: function() {
        console.log(1)
    } // 所有组件中create生命周期都会console1
}) // 接收到对象
```
4. 通过this.$options拿到new vue 时的参数

### 实现
vue.use()

> myrouter/index.js

```
class HistoryRoute {
  constructor () {
    this.current = null
  }
}

class VueRouter {
  constructor (options) {
    this.history = new HistoryRoute()
    this.mode = options.mode || 'hash'
    this.routes = options.routes || []
    this.routesMap = this.createMap(this.routes)
    this.init()
  }
  init () { // 监听事件的注册
    if (this.mode === 'hash') {
      // 判断有没有#，没有的话就自动加上#
      location.hash ? '' : location.hash = '/'
      // 监听页面loaded
      window.addEventListener('load', () => {
        // hash 取到 ‘#/../..’ 去掉#
        this.history.current = location.hash.slice(1)
      })
      // 监听hashchange
      window.addEventListener('hashchange', () => {
        // hash 取到 ‘#/../..’ 去掉#
        this.history.current = location.hash.slice(1)
      })
    } else if (this.mode === 'history') {
      location.pathname ? '' : location.pathname = '/'
      // 监听页面loaded
      window.addEventListener('load', () => {
        // pathneme不带# 直接赋值
        this.history.current = location.pathname
      })
      // 监听hashchange
      window.addEventListener('popstate', () => {
        // pathneme不带# 直接赋值
        this.history.current = location.pathname
      })
    }
  }
  createMap (routes) {
    // 把路由和组件匹配上
    // 这里构造路由表对象使用到了reduce方法，reduce方法是一个用于实现累加操作的方法，array.reduce(function(total, currentValue, currentIndex, arr), initialValue)，当传入了initialValue，那么total就会等于initialValue的值，currentValue就是数组的第一个元素，接着下一轮循环，会把函数的返回值当做total传入，数组的第二个元素当做currentValue传入，一直循环直到数组元素遍历完毕。
    return routes.reduce((memo, current) => {
      memo[current.path] = current.components
      return memo
    })
  }
}
// defineProperty()  是定义了一个属性标签，通过里面的get set 才能实现双向绑定
// install接收vue参数， 指向Vue的类，Vue的构造函数
VueRouter.install = function (Vue) {
  // 注入与混合
  Vue.mixin({
    beforeCreate () {
      // this.$options 是new vue 的时候传进的参数 el, router,store 等等
      // this.$options.router class vueRouter类的实例化对象
      if (this.$options && this.$options.router) {
        // 当前实例挂在 _root上
        this._root = this
        this._router = this.$options.router
        // vue 自己源码内部拿这个方法实现数据双向绑定，监听
        // this指向当前组件实例
        Vue.util.defineReactive(this, 'current', this._router.history) // defineReactive中调用defineProperty()
      }
      // 设置this._root的只可读的引用this.$router，只有get方法，没有set就不能对$router进行修改，可以this.$router = 134,但是不生效
      Object.defineProperty(this, '$router', {
        get () {
          return this._root._router
        }
      })
      // 当前路由信息 current
      // 设置this._root._router.history.current的只可读的引用this.$route，只有get方法，没有set就不能对$router进行修改，可以this.$router = 134,但是不生效
      Object.defineProperty(this, '$route', {
        get () {
          return this._root._router.history.current
        }
      })
    }
  })
  // 注册<router-view>
  Vue.component('router-view', {
    // r方法 作用 将接收到的字符串render成dom
    render (r) {
      // this 指向proxy (当前组件的代理对象)
      // this._self 指向当前组件的实例化对象
      // this._self._root 当前组件的对象
      let current = this._self._root._router.history.current // 拿到当前组件是哪个实例的路径
      let routeMap = this._self._root._router.history.current
      r(routeMap[current])
    }
  })
}
export default VueRouter

```
> router/index.js

```
import Vue from 'vue'
// import Router from 'vue-router'
import Router from '../myrouter'
import HelloWorld from '@/components/HelloWorld'
import Test from '@/components/test'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/test',
      name: 'Test',
      component: Test
    }
  ]
})
```


---
## 定义一个类，实例化后成一个对象