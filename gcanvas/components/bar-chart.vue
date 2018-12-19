<template>
  <gcanvas ref="canvas" class="wrapper"></gcanvas>
</template>

<script>
import { enable, WeexBridge } from 'gcanvas.js'
import F2 from './chart'

export default {
  name: 'bar-chart',
  data () {
    return {
      data: [
        {genre: '2018-03-05', sold: 275},
        {genre: '圆通', sold: 115},
        {genre: '装载率', sold: 120},
        {genre: 'Shooter', sold: 350},
        {genre: 'Other', sold: 150}
      ]
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
      chart.source(this.data)

      // Step 3：创建图形语法，绘制柱状图，由 genre 和 sold 两个属性决定图形位置，genre 映射至 x 轴，sold 映射至 y 轴
      chart.point().position('genre*sold').color('genre', (cvalue) => {
        if (cvalue === '圆通') {
          return 'red'
        }
        return 'green'
      }).shape('genre', (value) => {
        if (value === '圆通') {
          return 'circle'
        }
        return 'rect'
      }).size(50)
      chart.legend('genre', {
        marker: {
          radius: 6 // 半径大小
        }
      })
      chart.scale({
        genre: {
          alias: '类别' // 为属性定义别名
        },
        sold: {
          alias: '横坐标' // 为属性定义别名
        }
      })
      // Step 4: 渲染图表
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
<template>
  <gcanvas ref="canvas" class="wrapper"></gcanvas>
</template>

<script>
import { enable, WeexBridge } from 'gcanvas.js'
import F2 from './chart'

export default {
  name: 'bar-chart',
  data () {
    return {
      data: [
        {genre: '2018-03-05', sold: 275},
        {genre: '圆通', sold: 115},
        {genre: '装载率', sold: 120},
        {genre: 'Shooter', sold: 350},
        {genre: 'Other', sold: 150}
      ]
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
      chart.source(this.data)

      // Step 3：创建图形语法，绘制柱状图，由 genre 和 sold 两个属性决定图形位置，genre 映射至 x 轴，sold 映射至 y 轴
      chart.point().position('genre*sold').color('genre', (cvalue) => {
        if (cvalue === '圆通') {
          return 'red'
        }
        return 'green'
      }).shape('genre', (value) => {
        if (value === '圆通') {
          return 'circle'
        }
        return 'rect'
      }).size(50)
      chart.legend('genre', {
        marker: {
          radius: 6 // 半径大小
        }
      })
      chart.scale({
        genre: {
          alias: '类别' // 为属性定义别名
        },
        sold: {
          alias: '横坐标' // 为属性定义别名
        }
      })
      // Step 4: 渲染图表
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
