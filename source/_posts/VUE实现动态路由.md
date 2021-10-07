---
title: vue实现动态路由
date: 2021-09-20 10:19:23
tags:
- Vue
categories: Vue
author: zjk537
keywords: vue
description: vue实现动态路由
cover: 
top_img: 
---

### 为什么要实现动态路由?
我们在开发后台管理系统的过程中，会有不同的人来操作系统，有admin（管理员）、superAdmin(超管)，还会有各种运营人员、财务人员。为了区别这些人员，我们会给不同的人分配不一样的角色，从而来展示不同的菜单，这个就必须要通过动态路由来实现。
**主流的实现方式**
简单聊一下两种方式的优势，毕竟如果你从来没做过，说再多也看不明白，还是得看代码
**前端控制**
> 1. 不用后端帮助，路由表维护在前端
2. 逻辑相对比较简单，比较容易上手

**后端控制**
> 1. 相对更安全一点
2. 路由表维护在数据库

#### 一、前端控制
思路：在路由配置里，通过meta属性，扩展权限相关的字段，在路由守卫里通过判断这个权限标识，实现路由的动态增加，及页面跳转；如：我们增加一个role字段来控制角色
具体方案：
> 1、根据登录用户的账号，返回前端用户的角色
2、前端根据用户的角色，跟路由表的meta.role进行匹配
3、讲匹配到的路由形成可访问路由
**核心代码逻辑**
##### 1、在router.js文件（把静态路由和动态路由分别写在router.js）

```js
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

import Layout from '@/layout'

// constantRoutes  静态路由，主要是登录页、404页等不需要动态的路由
export const constantRoutes = [
  {
    path: '/redirect',
    component: Layout,
    hidden: true,
    children: [
      {
        path: '/redirect/:path*',
        component: () => import('@/views/redirect/index')
      }
    ]
  },
  {
    path: '/login',
    component: () => import('@/views/login/index'),
    hidden: true
  },
  {
    path: '/404',
    component: () => import('@/views/error-page/404'),
    hidden: true
  },
  {
    path: '/401',
    component: () => import('@/views/error-page/401'),
    hidden: true
  }
] 

// asyncRoutes 动态路由
export const asyncRoutes = [
  {
    path: '/permission',
    component: Layout,
    redirect: '/permission/page',
    alwaysShow: true, 
    name: 'Permission',
    meta: {
      title: 'Permission',
      icon: 'lock',
      // 核心代码，可以通过配的角色来进行遍历，从而是否展示
      // 这个意思就是admin、editor这两个角色，这个菜单是可以显示
      roles: ['admin', 'editor']
    },
    children: [
      {
        path: 'page',
        component: () => import('@/views/permission/page'),
        name: 'PagePermission',
        meta: {
          title: 'Page Permission',
          // 这个意思就是只有admin能展示
          roles: ['admin']
        }
      }
     ]
    }
]

const createRouter = () => new Router({
  scrollBehavior: () => ({ y: 0 }),
  routes: constantRoutes
})

const router = createRouter()

// 这个是重置路由用的，很有用，别看这么几行代码
export function resetRouter() {
  const newRouter = createRouter()
  router.matcher = newRouter.matcher 
}

export default router
```
##### 2、store/permission.js(在vuex维护一个state，通过配角色来控制菜单显不显示)

```js
import { asyncRoutes, constantRoutes } from '@/router'

// 这个方法是用来把角色和route.meta.role来进行匹配
function hasPermission(roles, route) {
  if (route.meta && route.meta.roles) {
    return roles.some(role => route.meta.roles.includes(role))
  } else {
    return true
  }
}


// 这个方法是通过递归来遍历路由，把有权限的路由给遍历出来
export function filterAsyncRoutes(routes, roles) {
  const res = []

  routes.forEach(route => {
    const tmp = { ...route }
    if (hasPermission(roles, tmp)) {
      if (tmp.children) {
        tmp.children = filterAsyncRoutes(tmp.children, roles)
      }
      res.push(tmp)
    }
  })

  return res
}

const state = {
  routes: [],
  addRoutes: []
}

const mutations = {
  SET_ROUTES: (state, routes) => {
    // 这个地方维护了两个状态一个是addRouters，一个是routes
    state.addRoutes = routes
    state.routes = constantRoutes.concat(routes)
  }
}

const actions = {
  generateRoutes({ commit }, roles) {
    return new Promise(resolve => {
      let accessedRoutes
      if (roles.includes('admin')) {
        accessedRoutes = asyncRoutes || []
      } else {
        // 核心代码，把路由和获取到的角色(后台获取的)传进去进行匹配
        accessedRoutes = filterAsyncRoutes(asyncRoutes, roles)
      }
      // 把匹配完有权限的路由给set到vuex里面
      commit('SET_ROUTES', accessedRoutes)
      resolve(accessedRoutes)
    })
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
}
```
#### 3、src/permission.js（新建一个路由守卫函数，可以在main.js，也可以抽离出来一个文件）
这里面的代码主要是控制路由跳转之前，先查一下有哪些可访问的路由，登录以后跳转的逻辑可以在这个地方写
```js
// permission.js
router.beforeEach((to, from, next) => {
  if (store.getters.token) { // 判断是否有token
    if (to.path === '/login') {
      next({ path: '/' });
    } else {
        // 判断当前用户是否已拉取完user_info信息
      if (store.getters.roles.length === 0) {
        store.dispatch('GetInfo').then(res => { // 拉取info
          const roles = res.data.role;
          // 把获取到的role传进去进行匹配，生成可以访问的路由
          store.dispatch('GenerateRoutes', { roles }).then(() => { 
            // 动态添加可访问路由表（核心代码，没有它啥也干不了）
            router.addRoutes(store.getters.addRouters)
            
            // hack方法 确保addRoutes已完成
            next({ ...to, replace: true })
          })
        }).catch(err => {
          console.log(err);
        });
      } else {
        next() //当有用户权限的时候，说明所有可访问路由已生成 如访问没权限的全面会自动进入404页面
      }
    }
  } else {
    if (whiteList.indexOf(to.path) !== -1) { // 在免登录白名单，直接进入
      next();
    } else {
      next('/login'); // 否则全部重定向到登录页
    }
  }
})
```
#### 4、侧边栏的可以从vuex里面取数据来进行渲染
核心代码是从router取可以用的路由对象，来进行侧边栏的渲染，不管是前端动态加载还是后端动态加载路由，这个代码都是一样的

```html
<!-- layout/components/siderbar.vue -->
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
    // 把取到的路由进行循环作为参数传给子组件
    <sidebar-item v-for="route in routes" :key="route.path" :item="route" :base-path="route.path" />
</el-menu>
// 获取有权限的路由
routes() {
  return this.$router.options.routes
}


<!-- layout/components/siderbarItem.vue -->
  <template slot="title">
    <item v-if="item.meta" :icon="item.meta && item.meta.icon" :title="item.meta.title" />
  </template>
  <sidebar-item
    v-for="child in item.children"
    :key="child.path"
    :is-nest="true"
    :item="child"
    :base-path="resolvePath(child.path)"
    class="nest-menu"
  />

  props: {
    // route object
    item: {
      type: Object,
      required: true
    },
    isNest: {
      type: Boolean,
      default: false
    },
    basePath: {
      type: String,
      default: ''
    }
  }

```
前端控制路由，逻辑相对简单，后端只需要存这个用户的角色就可以了，前端拿用户的角色进行匹配。但是如果新增角色，就会非常痛苦，每一个都要加。

### 二、后端控制路由
后端控制大致思路是：路由配置放在数据库表里，用户登录成功后，根据角色权限，把有权限的菜单传给前端，前端格式化成页面路由识别的结构，再渲染到页面菜单上；
> 1. 用户登录以后，后端根据该用户的角色，直接生成可访问的路由数据，注意这个地方是数据
2. 前端根据后端返回的路由数据，转成自己需要的路由结构

具体逻辑：
> 1. router.js里面只放一些静态的路由，login、404之类
2. 整理一份数据结构，存到表里
3. 从后端获取路由数据，写一个数据转换的方法，讲数据转成可访问的路由
4. 也是维护一个vuex状态，将转换好的路由存到vuex里面
5. 侧边栏也是从路由取数据进行渲染

因为前段控制和后端控制，后面的流程大部分都是一样的，所以这个地方只看看前面不一样的流程：
##### 1、store/permission.js，在vuex里面发送请求获取数据
```js
GenerateRoutes({ commit }, data) {
  return new Promise((resolve, reject) => {
    getRoute(data).then(res => {
     // 将获取到的数据进行一个转换，然后存到vuex里
      const accessedRouters = arrayToMenu(res.data)
      accessedRouters.concat([{ path: '*', redirect: '/404', hidden: true }])
      commit('SET_ROUTERS', accessedRouters)
      resolve()
    }).catch(error => {
      reject(error)
    })
  })
}
```
##### 2、整理一份数据结构，存到表里
```js
// 页面路由格式
{
    path: '/form',
    component: Layout,
    children: [
      {
        path: 'index',
        name: 'Form',
        component: () => import('@/views/form/index'),
        meta: { title: 'Form', icon: 'form' }
      }
    ]
}

// 整理后的数据格式
// 一级菜单
// parentId为0的就可以当做一级菜单，id最好是可以选4位数，至于为什么等你开发项目的时候就知道了
{
    id: 1300
    parentId: 0
    title: "菜单管理"
    path: "/menu"
    hidden: false
    component: null
    hidden: false
    name: "menu"
}，

// 二级菜单
// parentId不为0的，就可以拿parentId跟一级菜单的id去匹配，匹配上的就push到children里面
{
    id: 1307
    parentId: 1300
    title: "子菜单"
    hidden: false
    path: "menuItem"
    component: "menu/menuItem" // 要跟本地的文件地址匹配上
    hidden: false
    name: "menuItem"
}

```
##### 3、写一个转化方法，把获取到的数据转换成router结构
```js
export function arrayToMenu(array) {
  const nodes = []
  // 获取顶级节点
  for (let i = 0; i < array.length; i++) {
    const row = array[i]
    // 这个exists方法就是判断下有没有子级
    if (!exists(array, row.parentId)) {
      nodes.push({
        path: row.path, // 路由地址
        hidden: row.hidden, // 全部true就行，如果后端没配
        component: Layout, // 一般就是匹配你文件的component
        name: row.name, // 路由名称
        meta: { title: row.title, icon: row.name }, // title就是显示的名字
        id: row.id, // 路由的id
        redirect: 'noredirect'
      })
    }
  }
  const toDo = Array.from(nodes)
  while (toDo.length) {
    const node = toDo.shift()
    // 获取子节点
    for (let i = 0; i < array.length; i++) {
      const row = array[i]
      // parentId等于哪个父级的id，就push到哪个
      if (row.parentId === node.id) {
        const child = {
          path: row.path,
          name: row.name,
          hidden: row.hidden,
          // 核心代码，因为二级路由的component是需要匹配页面的
          component: require('@/views/' + row.component + '/index.vue'),
          meta: { title: row.title, icon: row.name },
          id: row.id
        }
        if (node.children) {
          node.children.push(child)
        } else {
          node.children = [child]
        }
        toDo.push(child)
      }
    }
  }
  return nodes
}
// 看下有没有子级
function exists(rows, parentId) {
  for (let i = 0; i < rows.length; i++) {
    if (rows[i].id === parentId) return true
  }
  return false
}
```