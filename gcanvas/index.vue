<template>
  <div class="wrapper">
    <scroller style="flex: 1; width: 750px;">
      <pie-chart></pie-chart>
      <bar-chart></bar-chart>
      <line-chart></line-chart>
    </scroller>
  </div>
</template>
<script>
import BarChart from './components/bar-chart'
import LineChart from './components/line-chart'
import PieChart from './components/pie-chart'
const navigator = weex.requireModule('navigator')

export default {
  data () {
    return {
      chart: null,
      timeStamp: 0
    }
  },
  components: { PieChart, LineChart, BarChart },
  methods: {
    leftClick () {
      navigator.pop()
    }
  }
}
</script>
<style scoped>
  .wrapper {
    width: 750px;
    background-color: white;
  }
</style>
