<template>
  <div class="tabs-wrap">
    <ul class='tab-wrap' v-on:click='handleClick' :style='transition'>
      <li v-for='(item, index) in list' :data-index='index' :class='{active: (cur==index)}'>{{item}}</li>
    </ul>
    <div class="slider" :style='`transform:translatex(${sliderX}px) scale(${sliderScale});width:${sliderW}px;`'></div>
    <ul class='content-wrap' :style='`transform:translatex(${contentX}px);width:${list.length * containerW}px;`' v-on:touchstart='handleTouchStart' v-on:touchmove='handleTouchMove' v-on:touchend='handleTouchEnd' v-on:touchcancel='handleTouchCancel'>
      <slot></slot>
    </ul>
  </div>
</template>

<script>
const EFFECT_DISTANCE = 30
const END_TIME = 200
const SCALE = 0.3

var startX = 0
var startY = 0
var moveX = 0
var moveY = 0
var moveOffsetX = 0
var moveOffsetY = 0
var totalOffsetX = 0
var isStarted = false
var clientX = 0
var clientY = 0
var rafReqId = null

var isEndAnimating = false

export default {
  name: 'Vue-Tabs',
  props: {
    list: {
      type: Array,
      required: true
    },
    sliderW: {
      type: Number,
      default: 100
    },
    callback: {
      type: Function,
      default: function () {}
    }
  },
  data () {
    return {
      containerW: window.screen.availWidth < 300 ? 360 : window.screen.availWidth,
      cur: 0,
      sliderCls: '',
      animateTime: 0,
      transition: '',
      sliderX: 0,
      contentX: 0,
      sliderScale: 1
    }
  },
  computed: {
    tabItemW () {
      return this.containerW / this.list.length
    },
    wRate () {
      return 1 / this.list.length
    },
    sliderInitX () {
      return (this.tabItemW - this.sliderW) / 2
    },
    halfContentW () {
      return this.containerW / 2
    }
  },
  methods: {
    doAnimationFrame (interval, fn, completeFn) {
      var raf = window.requestAnimationFrame || window.webkitRequestAnimationFrame
      var cancelRaf = window.cancelAnimationFrame
      if (rafReqId) {
        cancelRaf(rafReqId)
      }
      var start = window.performance.now()
      var offset = 0

      var step = function (timestamp) {
        offset = timestamp - start
        if (offset <= interval) {
          fn(offset, interval)
          rafReqId = raf(step)
        } else {
          fn(interval, interval)
          completeFn()
          rafReqId = null
        }
      }
      rafReqId = raf(step)
    },
    handleClick (e) {
      var index = parseInt(e.target.getAttribute('data-index'))
      if (isNaN(index) || index === this.cur) {
        return
      }
      // var dif = index - this.cur
      // var conDif = this.containerW * dif
      var conDif = this.containerW * index - Math.abs(this.contentX)
      // var tabDif = this.tabItemW * dif
      var tabDif = this.tabItemW * index - Math.abs(this.sliderX)
      var sliderX = this.sliderX
      var contentX = this.contentX
      this.cur = index
      this.callback(index)
      isEndAnimating = true
      this.doAnimationFrame(this.animateTime || END_TIME, (offset, interval) => {
        var r = offset / interval
        var rDif = r * conDif
        this.setContentX(contentX - rDif)
        this.setSliderX(sliderX + r * tabDif + this.sliderInitX)
        this.doSliderScale(rDif % this.containerW)
      }, function () {
        isEndAnimating = false
      })
    },
    setSliderScale (scale) {
      this.sliderScale = scale
    },
    setSliderX (moveOffsetX) {
      this.sliderX = moveOffsetX
    },
    setContentX (moveOffsetX) {
      this.contentX = moveOffsetX
    },
    reset () {
      isStarted = false
      moveOffsetX = 0
      moveOffsetY = 0
      totalOffsetX = 0
    },
    addEndAnimation (contentMoveOffsetX) {
      var sliderX = this.sliderX
      var contentX = this.contentX
      this.doAnimationFrame(this.animateTime || END_TIME, (offset, interval) => {
        var r = offset / interval
        var contentF = contentX + r * contentMoveOffsetX
        this.setContentX(contentF)
        this.setSliderX(sliderX - r * (contentMoveOffsetX * this.wRate))
        this.doSliderScale(contentF % this.containerW)
      }, function () {
        isEndAnimating = false
      })
    },
    doSliderScale (contentOffset) {
      contentOffset = Math.abs(contentOffset)
      var scale = 1
      if (this.halfContentW > contentOffset) {
        scale = 1 + contentOffset / this.halfContentW * SCALE
      } else {
        scale = (1 + SCALE) - (contentOffset - this.halfContentW) / this.halfContentW * SCALE
      }
      this.setSliderScale(scale)
    },
    doEndAnimate (offset) {
      if (offset === 0) {
        return
      }
      isEndAnimating = true
      if (Math.abs(offset) > EFFECT_DISTANCE) {
        if (offset > 0) { // 右移
          this.addEndAnimation(this.containerW - offset)
          this.cur--
        } else { // 左移
          this.addEndAnimation(-this.containerW - offset)
          this.cur++
        }
        this.callback(this.cur)
      } else { // 回滚
        this.addEndAnimation(-offset)
      }
    },
    doEnd () {
      this.doEndAnimate(totalOffsetX)
    },
    handleTouchStart (event) {
      isStarted = true
      startX = event.touches[0].clientX
      startY = event.touches[0].clientY
    },
    handleTouchMove (event) {
      if (!isStarted || isEndAnimating) {
        return
      }
      var isFirst = (moveOffsetX === 0 && moveOffsetY === 0)
      clientX = event.touches[0].clientX
      clientY = event.touches[0].clientY
      moveOffsetX = clientX - (moveX || startX)
      moveOffsetY = clientY - (moveY || startY)
      moveX = clientX
      moveY = clientY
      if (isFirst) { // 第一次move,仅做方向判断，不移动
        if (Math.abs(moveOffsetX) < Math.abs(moveOffsetY)) { // 上下优先
          this.reset()
          return
        }
      } else {
        if ((this.cur === 0 && moveOffsetX > 0) || (this.cur === this.list.length - 1 && moveOffsetX < 0)) {
          return
        }
        totalOffsetX += moveOffsetX
        this.setContentX(this.contentX + moveOffsetX)
        this.setSliderX(this.sliderX - moveOffsetX * this.wRate)
        this.doSliderScale(totalOffsetX)
      }
      event.preventDefault()
      event.stopPropagation()
    },
    handleTouchEnd () {
      if (!isStarted) {
        return
      }
      this.doEnd()
      this.reset()
    },
    handleTouchCancel () {
      if (!isStarted) {
        return
      }
      this.doEnd()
      this.reset()
    }
  },
  mounted () {
    this.setSliderX(this.sliderInitX)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  li{
    list-style-type: none;
    flex: 1;
    text-align: center;
  }
  ul{
    display: flex;
    margin: 0;
    padding: 0;
    will-change: transform;
  }
  .slider{
    height: 1px;
    background-color: #03A9F4;
    will-change: transform;
  }
  .tabs-wrap{
    width: 100%;
    overflow: hidden;
  }
</style>
