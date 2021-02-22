我们常用的加载组件的方式就是 import * from '../src'，即加载项目中的组件。
今天带来一种方式可以通过HTTP加载远程组件
适用场景
需要引入第三方开发的组件
比如自己开发的项目，需要加载第三方开发的组件，此时可以通过这种方式进行。
第三方只需要开发自己的组件，然后打包成我们需要的组件，然后发布，然后我们这边引入即可
完全通过接口生成页面
完全通过接口配置页面，每个组件都通过http方式远程加载，实现页面的配置效果，从而达到可以任意拼接页面。通过减小组件颗粒度，提升组件的高复用性，减少测试时间
实现
组件打包
vue-cli文档中单文件组件打包
此处正式使用此命令行，对单文件组件进行打包
build: "vue-cli-service build --target lib --formats umd-min  --name 'mincc' ./src/components/HelloWorld.vue"
// --name 后面为 组件名
// 最后的./src/components/HelloWorld.vue为组件路径
// --formats 后面为 打包的格式
即在打包的时候，打包单文件为 umd-min 模式，需要在 vue.config.js 中配置 css: { extract: false } 使 js与css 打包为一个文件
加载组件
正常来说，加载组件都是使用 import，但是加载远程组件
// 并不生效，import是无法加载外部资源的
const MyComponent = () => import('https://**/**.js')
所以我们可以通过加载script 标签的方式，加载外部资源
export default async function externalComponent(url) {
  const name = url.split('/').reverse()[0].match(/^(.*?)\.umd/)[1]
  if (window[name]) return window[name]
  window[name] = new Promise((resolve, reject) => {
    const script = document.createElement('script')
    script.async = true
    script.addEventListener('load', () => {
      resolve(window[name])
    })
    script.addEventListener('error', () => {
      reject(new Error(`Error loading ${url}`))
    })
    script.src = url
    document.head.appendChild(script)
  })
  return window[name]
}
usage
<template>
  <div id="app">
    <div :is="com"></div>
  </div>
</template>
<script>
import externalComponent from '@/utils/loadExternalComponent.js'
export default {
  name: 'App',
  data () {
    return {
      com: null
    }
  },
  async created () {
    let com = await externalComponent('http://10.26.115.67:8899/mincc.umd.min.js')
    this.com = com
  }
}
</script>
拓展
通过接口的方式，返回页面的结构配置，达到通过配置生成页面的方法
服务端
a. 用于生成组件的上传&接收&保存（版本等可以在此通过存储文件夹标记
b. 请求组件的时候充当静态服务器
c. 存储配置好页面的结构    例如页面由组件 （a, c, d 组成
d. 分用户 和 权限
组件生成平台
     打包组件，发布时配置好 （版本&发布地址&用户名&密码等参数
页面基座
在线组件 直接通过接口加载远程组件生成页面
离线组件 下载远程组件至本地项目中，通过import方式，拼接组件为页面
