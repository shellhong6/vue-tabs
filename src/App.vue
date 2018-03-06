<template>
  <div id="app">
    <VueTabs :list='list' :slider-w='80'>
      <li class="red">
        111111<br>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
      </li>
      <li class="green">
        222222<br>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
      </li>
      <li class="blue">
        333333<br>
        333333<br>
        333333<br>
        333333<br>
        333333<br>
        333333<br>
        333333<br>
        333333<br>
      </li>
    </VueTabs>
  </div>
</template>

<script>
import VueTabs from './components/Vue-Tabs'

export default {
  name: 'app',
  data () {
    return {
      list: ['全部', '收入', '支出']
    }
  },
  components: {
    VueTabs
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.red {
  background: red;
}
.green {
  background: green;
}
.blue {
  background: blue;
}
</style>
