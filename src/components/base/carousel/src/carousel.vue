<template>
  <article class="email-bind-carousel">
    <slider :pages="someList" :sliderinit="sliderinit" @slide='slide'>
    <!-- slot  -->
    </slider>
  </article>
</template>

<script>
import slider from '../../slider';
  export default {
  data () {
    return {
      someList: [{}, {}, {}],
      sliderinit: {
        currentPage: 0,
        thresholdTime: 500, // 滑动时间阈值判定距离
        thresholdDistance: 100 // 滑动距离阈值
        // slideSpeed:1000, // 滑动速度
        // loop:true, // 无限循环
        // autoplay:1000 // 自动播放:时间[ms]
      }
    }
  },
  components: {
    slider
  },
  methods: {
    turnTo (num) {
      // 传递事件 vue 2.0 传递事件修改了，好的写法应该直接写在空vue类中
      this.$children[0].$emit('slideTo', num)
    },
    slideNext () {
      this.$children[0].$emit('slideNext')
    },
    slidePre () {
      this.$children[0].$emit('slidePre')
    },
    appendslider () {
      this.someList.push({
        title: 'slidernew',
        style: {
          background: '#333',
          color: '#fff'
        }
      })
    },
    // 监听事件发生了变化,需要指向一个子组件实例
    slide (pagenum) {
      
    }
  }
}
</script>

<style lang="less">

</style>