# Vue-Tabs

> 一个基于vue2.x的移动端tabs切换组件

[demo](http://blog.shellhong.com/effect/vue-tabs/index.html)


## Build Setup

> 构建基于vue-cli

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

## 组件使用实例

```
<template>
  <div id="app">
    <VueTabs :list='list'>
      <li>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
        111111<br>
      </li>
      <li>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
        222222<br>
      </li>
      <li>
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
```
