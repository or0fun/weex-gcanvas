<template>
  <gcanvas ref="canvas" class="wrapper"></gcanvas>
</template>

<script>
import { enable, WeexBridge } from 'gcanvas.js'
import F2 from './chart'

export default {
  name: 'line-chart',
  data () {
    return {
      data: [{
        time: '2016-08-08 00:00:00',
        tem: 10
      }, {
        time: '2016-08-08 00:10:00',
        tem: 22
      }, {
        time: '2016-08-08 00:30:00',
        tem: 20
      }, {
        time: '2016-08-09 00:35:00',
        tem: 26
      }, {
        time: '2016-08-09 01:00:00',
        tem: 20
      }, {
        time: '2016-08-09 01:20:00',
        tem: 26
      }, {
        time: '2016-08-10 01:40:00',
        tem: 28
      }, {
        time: '2016-08-10 02:00:00',
        tem: 20
      }, {
        time: '2016-08-10 02:20:00',
        tem: 18
      }]
    }
  },
  mounted () {
    this.draw()
  },
  methods: {
    draw () {
      let ref = this.$refs.canvas
      ref = enable(ref, {bridge: WeexBridge})
      let ctx = ref.getContext('2d')
      const canvas = new F2.Renderer(ctx)
      const chart = new F2.Chart({
        el: canvas, // 将第三步创建的 canvas 对象的上下文传入
        width: 750, // 必选，图表宽度，同 canvas 的宽度相同
        height: 400 // 必选，图表高度，同 canvas 的高度相同
      })
      this.chart = chart
      let defs = {
        time: {
          type: 'timeCat',
          mask: 'MM/DD',
          range: [0, 1]
        },
        tem: {
          tickCount: 5,
          min: 0,
          alias: '日均温度'
        }
      }
      chart.source(this.data, defs)
      chart.axis('time', {
        label: (text, index, total) => {
          const textCfg = {}
          if (index === 0) {
            textCfg.textAlign = 'left'
          } else if (index === total - 1) {
            textCfg.textAlign = 'right'
          }
          return textCfg
        }
      })
      chart.tooltip({
        showCrosshairs: true
      })
      chart.line().position('time*tem').shape('smooth').size(4)
      chart.point().position('time*tem').shape('smooth').size(5).style({
        stroke: '#fff',
        lineWidth: 2
      })
      chart.render()
    }
  }
}
</script>

<style scoped>
  .wrapper {
    width: 750px;
    height: 400px;
  }
</style>
