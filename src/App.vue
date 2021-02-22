<template>
  <div id="app">
    <div :is="com"></div>
    <!-- <div :is="hello"></div> -->
  </div>
</template>

<script>
import { externalComponent } from '@buera/handyjs'
import hello from '@/components/HelloWorld.vue'
export default {
  name: 'App',
  data () {
    return {
      com: null,
      hello
    }
  },
  async created () {
    let com = await externalComponent('http://10.26.115.67:8899/mincc.umd.min.js')
    this.com = com
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
