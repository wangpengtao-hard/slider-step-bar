<template>
  <div id="app">
    <Drag @confirm="aaa"
     ref="drag" />
    <!-- <MobileDrag
      label="岳林书院-2层-10人（电视、投影）"
      :blockWidth="32"
      :blockHeight="32"
      @confirm="aaa"
    ></MobileDrag> -->
  </div>
</template>

<script>
import Drag from './components/Drag.vue'
// import MobileDrag from './components/MobileDrag.vue'

export default {
  name: 'App',
  data() {
    return {
    }
  },
  components: {
    // MobileDrag
    Drag
  },
  methods: {
    aaa(a) {
      console.log(a)
      // setTimeout(() => {
      // }, 3000);
      // this.$refs['drag'].reset();
    }
  }
}
</script>

<style>
body {
  margin: 20px;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 1000px;
  width: 1200px;
  /* overflow: auto; */
}
</style>
