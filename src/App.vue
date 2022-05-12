<template>
  <div id="app">
    <HelloWorld  msg="Welcome to Your Vue.js App"/>
    <Pie/>
    <Edge />
    <Test/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Pie from './components/Pie.vue'
import Edge from './components/Edge.vue'
import Test from './components/Test.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    Edge,
    Pie,
    Test
  }
}
</script>

<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
