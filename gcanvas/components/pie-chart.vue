<template>
  <gcanvas ref="canvas" class="wrapper"></gcanvas>
</template>

<script>
import { enable, WeexBridge } from 'gcanvas.js'
import F2 from './chart'

export default {
  name: 'pie-chart',
  data () {
    return {
      data: [{
        name: '老客户数',
        percent: 0.3654,
        a: '1'
      }, {
        name: '新客户数',
        percent: 0.6346,
        a: '1'
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

      chart.source(this.data, {
        percent: {
          formatter: function formatter (val) {
            return val * 100 + '%'
          }
        }
      })
      chart.tooltip(false)
      chart.coord('polar', {
        transposed: true,
        radius: 0.85,
        innerRadius: 0.8
      })
      chart.axis(false)
      chart.interval().position('a*percent').color('name', ['#3388FF', '#FFB946']).adjust('stack').style({
        lineWidth: 3,
        stroke: '#fff',
        lineJoin: 'round',
        lineCap: 'round'
      })
      chart.guide().text({
        position: ['50%', '45%'],
        content: '45360',
        style: {
          fill: '#333333',
          textAlign: 'center',
          fontSize: 32
        }
      })
      chart.guide().text({
        position: ['50%', '60%'],
        content: '付款人数',
        style: {
          fill: '#999999',
          textAlign: 'center',
          fontSize: 28
        }
      })
      chart.render()
      this.addPieLabel(chart, chart.get('canvas'))
    },

    drawLabel (chart, label, labelGroup) {
      const APPEND_OFFSET = 30
      const canvasWidth = chart.get('width')

      let _data = label._data
      let _anchor = label._anchor
      let _router = label._router
      let fill = label.fill
      let y = label.y

      let labelAttrs = {
        y: y,
        fontSize: 24, // 字体大小
        fill: '#808080',
        text: _data.name + '\n' + _data.percent.toFixed(2),
        textBaseline: 'middle'
      }
      let lastPoint = {
        y: y
      }

      if (label._side === 'left') {
        // 具体文本的位置
        lastPoint.x = APPEND_OFFSET
        labelAttrs.x = APPEND_OFFSET // 左侧文本左对齐并贴着画布最左侧边缘
        labelAttrs.textAlign = 'left'
      } else {
        lastPoint.x = canvasWidth - APPEND_OFFSET
        labelAttrs.x = canvasWidth - APPEND_OFFSET // 右侧文本右对齐并贴着画布最右侧边缘
        labelAttrs.textAlign = 'right'
      }

      // 绘制文本
      labelGroup.addShape('Text', {
        attrs: labelAttrs
      })
      // labels.push(text)

      // 绘制锚点
      // labelGroup.addShape('Circle', {
      //   attrs: {
      //     x: _anchor.x,
      //     y: _anchor.y,
      //     r: 2,
      //     fill: fill
      //   }
      // })
      // 绘制连接线
      let points = void 0
      if (_router.y !== y) {
        // 文本位置做过调整
        points = [_anchor, {
          x: _router.x,
          y: y
        }, lastPoint]
      } else {
        points = [_anchor, _router, lastPoint]
      }

      labelGroup.addShape('Polyline', {
        attrs: {
          points: points,
          lineWidth: 1,
          stroke: fill
        }
      })
    },
    getEndPoint (center, angle, r) {
      return {
        x: center.x + r * Math.cos(angle),
        y: center.y + r * Math.sin(angle)
      }
    },
    getR (chart) {
      const coord = chart.get('coord') // 获取坐标系对象
      const r = coord.circleRadius // 极坐标半径
      return r
    },
    getCenter (chart) {
      const coord = chart.get('coord') // 获取坐标系对象
      const center = coord.center // 极坐标圆心坐标
      return center
    },
    addPieLabel (chart, canvas) {
      const coord = chart.get('coord') // 获取坐标系对象
      const center = coord.center // 极坐标圆心坐标
      const r = coord.circleRadius // 极坐标半径
      const LINEHEIGHT = 32
      const canvasHeight = chart.get('height')

      const ANCHOR_OFFSET = 5
      const OFFSET = 15
      let halves = [[], []]
      const labelGroup = canvas.addGroup()
      labelGroup && labelGroup.clear()
      let geom = chart.get('geoms')[0]
      let shapes = geom.get('container').get('children')
      shapes.forEach((shape) => {
        let shapeAttrs = shape.attr()
        let origin = shape.get('origin')
        let startAngle = shapeAttrs.startAngle
        let endAngle = shapeAttrs.endAngle

        let middleAngle = (startAngle + endAngle) / 2
        let edgePoint = this.getEndPoint(center, middleAngle, r + ANCHOR_OFFSET)
        let routerPoint = this.getEndPoint(center, middleAngle, r + OFFSET)
        let label = {
          _anchor: edgePoint,
          _router: routerPoint,
          _data: origin._origin,
          x: routerPoint.x,
          y: routerPoint.y,
          r: r + OFFSET,
          fill: origin.color // 字体颜色
        }
        // 判断文本的方向
        if (edgePoint.x < center.x) {
          label._side = 'left'
          halves[0].push(label)
        } else {
          label._side = 'right'
          halves[1].push(label)
        }
      })

      let maxCountForOneSide = parseInt(canvasHeight / LINEHEIGHT, 10)

      console.log(JSON.stringify(halves))
      halves.forEach((half, index) => {
        if (half.length > maxCountForOneSide) {
          half.sort((a, b) => {
            return b._percent - a._percent
          })
          half.splice(maxCountForOneSide, half.length - maxCountForOneSide)
        }

        half.sort((a, b) => {
          return a.y - b.y
        })
        this.antiCollision(chart, half, index, labelGroup)
        canvas.draw()
      })
    },
    antiCollision (chart, half, isRight, labelGroup) {
      const LINEHEIGHT = 32
      const OFFSET = 15
      const r = this.getR(chart)
      const center = this.getCenter(chart)
      const canvasHeight = chart.get('height')
      const startY = center.y - r - OFFSET - LINEHEIGHT
      let overlapping = true
      let totalH = canvasHeight
      let i = void 0

      let maxY = 0
      let minY = Number.MIN_VALUE
      const boxes = half.map((label) => {
        const labelY = label.y
        if (labelY > maxY) {
          maxY = labelY
        }
        if (labelY < minY) {
          minY = labelY
        }
        return {
          size: LINEHEIGHT,
          targets: [labelY - startY]
        }
      })
      if (maxY - startY > totalH) {
        totalH = maxY - startY
      }

      while (overlapping) {
        boxes.forEach((box) => {
          const target = (Math.min.apply(minY, box.targets) + Math.max.apply(minY, box.targets)) / 2
          box.pos = Math.min(Math.max(minY, target - box.size / 2), totalH - box.size)
        })

        // detect overlapping and join boxes
        overlapping = false
        i = boxes.length
        while (i--) {
          if (i > 0) {
            const previousBox = boxes[i - 1]
            const box = boxes[i]
            if (previousBox.pos + previousBox.size > box.pos) {
              // overlapping
              previousBox.size += box.size
              previousBox.targets = previousBox.targets.concat(box.targets)

              // overflow, shift up
              if (previousBox.pos + previousBox.size > totalH) {
                previousBox.pos = totalH - previousBox.size
              }
              boxes.splice(i, 1)
              overlapping = true
            }
          }
        }
      }

      // step 4: normalize y and adjust x
      i = 0
      boxes.forEach((b) => {
        let posInCompositeBox = startY
        b.targets.forEach(() => {
          half[i].y = b.pos + posInCompositeBox + LINEHEIGHT / 2
          posInCompositeBox += LINEHEIGHT
          i++
        })
      })
      // (x - cx)^2 + (y - cy)^2 = totalR^2
      half.forEach((label) => {
        const rPow2 = label.r * label.r
        const dyPow2 = Math.pow(Math.abs(label.y - center.y), 2)
        if (rPow2 < dyPow2) {
          label.x = center.x
        } else {
          const dx = Math.sqrt(rPow2 - dyPow2)
          if (!isRight) {
            // left
            label.x = center.x - dx
          } else {
            // right
            label.x = center.x + dx
          }
        }
        this.drawLabel(chart, label, labelGroup)
      })
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
  name: 'pie-chart',
  data () {
    return {
      data: [{
        name: '老客户数',
        percent: 0.3654,
        a: '1'
      }, {
        name: '新客户数',
        percent: 0.6346,
        a: '1'
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

      chart.source(this.data, {
        percent: {
          formatter: function formatter (val) {
            return val * 100 + '%'
          }
        }
      })
      chart.tooltip(false)
      chart.coord('polar', {
        transposed: true,
        radius: 0.85,
        innerRadius: 0.8
      })
      chart.axis(false)
      chart.interval().position('a*percent').color('name', ['#3388FF', '#FFB946']).adjust('stack').style({
        lineWidth: 3,
        stroke: '#fff',
        lineJoin: 'round',
        lineCap: 'round'
      })
      chart.guide().text({
        position: ['50%', '45%'],
        content: '45360',
        style: {
          fill: '#333333',
          textAlign: 'center',
          fontSize: 32
        }
      })
      chart.guide().text({
        position: ['50%', '60%'],
        content: '付款人数',
        style: {
          fill: '#999999',
          textAlign: 'center',
          fontSize: 28
        }
      })
      chart.render()
      this.addPieLabel(chart, chart.get('canvas'))
    },

    drawLabel (chart, label, labelGroup) {
      const APPEND_OFFSET = 30
      const canvasWidth = chart.get('width')

      let _data = label._data
      let _anchor = label._anchor
      let _router = label._router
      let fill = label.fill
      let y = label.y

      let labelAttrs = {
        y: y,
        fontSize: 24, // 字体大小
        fill: '#808080',
        text: _data.name + '\n' + _data.percent.toFixed(2),
        textBaseline: 'middle'
      }
      let lastPoint = {
        y: y
      }

      if (label._side === 'left') {
        // 具体文本的位置
        lastPoint.x = APPEND_OFFSET
        labelAttrs.x = APPEND_OFFSET // 左侧文本左对齐并贴着画布最左侧边缘
        labelAttrs.textAlign = 'left'
      } else {
        lastPoint.x = canvasWidth - APPEND_OFFSET
        labelAttrs.x = canvasWidth - APPEND_OFFSET // 右侧文本右对齐并贴着画布最右侧边缘
        labelAttrs.textAlign = 'right'
      }

      // 绘制文本
      labelGroup.addShape('Text', {
        attrs: labelAttrs
      })
      // labels.push(text)

      // 绘制锚点
      // labelGroup.addShape('Circle', {
      //   attrs: {
      //     x: _anchor.x,
      //     y: _anchor.y,
      //     r: 2,
      //     fill: fill
      //   }
      // })
      // 绘制连接线
      let points = void 0
      if (_router.y !== y) {
        // 文本位置做过调整
        points = [_anchor, {
          x: _router.x,
          y: y
        }, lastPoint]
      } else {
        points = [_anchor, _router, lastPoint]
      }

      labelGroup.addShape('Polyline', {
        attrs: {
          points: points,
          lineWidth: 1,
          stroke: fill
        }
      })
    },
    getEndPoint (center, angle, r) {
      return {
        x: center.x + r * Math.cos(angle),
        y: center.y + r * Math.sin(angle)
      }
    },
    getR (chart) {
      const coord = chart.get('coord') // 获取坐标系对象
      const r = coord.circleRadius // 极坐标半径
      return r
    },
    getCenter (chart) {
      const coord = chart.get('coord') // 获取坐标系对象
      const center = coord.center // 极坐标圆心坐标
      return center
    },
    addPieLabel (chart, canvas) {
      const coord = chart.get('coord') // 获取坐标系对象
      const center = coord.center // 极坐标圆心坐标
      const r = coord.circleRadius // 极坐标半径
      const LINEHEIGHT = 32
      const canvasHeight = chart.get('height')

      const ANCHOR_OFFSET = 5
      const OFFSET = 15
      let halves = [[], []]
      const labelGroup = canvas.addGroup()
      labelGroup && labelGroup.clear()
      let geom = chart.get('geoms')[0]
      let shapes = geom.get('container').get('children')
      shapes.forEach((shape) => {
        let shapeAttrs = shape.attr()
        let origin = shape.get('origin')
        let startAngle = shapeAttrs.startAngle
        let endAngle = shapeAttrs.endAngle

        let middleAngle = (startAngle + endAngle) / 2
        let edgePoint = this.getEndPoint(center, middleAngle, r + ANCHOR_OFFSET)
        let routerPoint = this.getEndPoint(center, middleAngle, r + OFFSET)
        let label = {
          _anchor: edgePoint,
          _router: routerPoint,
          _data: origin._origin,
          x: routerPoint.x,
          y: routerPoint.y,
          r: r + OFFSET,
          fill: origin.color // 字体颜色
        }
        // 判断文本的方向
        if (edgePoint.x < center.x) {
          label._side = 'left'
          halves[0].push(label)
        } else {
          label._side = 'right'
          halves[1].push(label)
        }
      })

      let maxCountForOneSide = parseInt(canvasHeight / LINEHEIGHT, 10)

      console.log(JSON.stringify(halves))
      halves.forEach((half, index) => {
        if (half.length > maxCountForOneSide) {
          half.sort((a, b) => {
            return b._percent - a._percent
          })
          half.splice(maxCountForOneSide, half.length - maxCountForOneSide)
        }

        half.sort((a, b) => {
          return a.y - b.y
        })
        this.antiCollision(chart, half, index, labelGroup)
        canvas.draw()
      })
    },
    antiCollision (chart, half, isRight, labelGroup) {
      const LINEHEIGHT = 32
      const OFFSET = 15
      const r = this.getR(chart)
      const center = this.getCenter(chart)
      const canvasHeight = chart.get('height')
      const startY = center.y - r - OFFSET - LINEHEIGHT
      let overlapping = true
      let totalH = canvasHeight
      let i = void 0

      let maxY = 0
      let minY = Number.MIN_VALUE
      const boxes = half.map((label) => {
        const labelY = label.y
        if (labelY > maxY) {
          maxY = labelY
        }
        if (labelY < minY) {
          minY = labelY
        }
        return {
          size: LINEHEIGHT,
          targets: [labelY - startY]
        }
      })
      if (maxY - startY > totalH) {
        totalH = maxY - startY
      }

      while (overlapping) {
        boxes.forEach((box) => {
          const target = (Math.min.apply(minY, box.targets) + Math.max.apply(minY, box.targets)) / 2
          box.pos = Math.min(Math.max(minY, target - box.size / 2), totalH - box.size)
        })

        // detect overlapping and join boxes
        overlapping = false
        i = boxes.length
        while (i--) {
          if (i > 0) {
            const previousBox = boxes[i - 1]
            const box = boxes[i]
            if (previousBox.pos + previousBox.size > box.pos) {
              // overlapping
              previousBox.size += box.size
              previousBox.targets = previousBox.targets.concat(box.targets)

              // overflow, shift up
              if (previousBox.pos + previousBox.size > totalH) {
                previousBox.pos = totalH - previousBox.size
              }
              boxes.splice(i, 1)
              overlapping = true
            }
          }
        }
      }

      // step 4: normalize y and adjust x
      i = 0
      boxes.forEach((b) => {
        let posInCompositeBox = startY
        b.targets.forEach(() => {
          half[i].y = b.pos + posInCompositeBox + LINEHEIGHT / 2
          posInCompositeBox += LINEHEIGHT
          i++
        })
      })
      // (x - cx)^2 + (y - cy)^2 = totalR^2
      half.forEach((label) => {
        const rPow2 = label.r * label.r
        const dyPow2 = Math.pow(Math.abs(label.y - center.y), 2)
        if (rPow2 < dyPow2) {
          label.x = center.x
        } else {
          const dx = Math.sqrt(rPow2 - dyPow2)
          if (!isRight) {
            // left
            label.x = center.x - dx
          } else {
            // right
            label.x = center.x + dx
          }
        }
        this.drawLabel(chart, label, labelGroup)
      })
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
