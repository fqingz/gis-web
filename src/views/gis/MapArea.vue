<template>
  <div>
    <div :style="{width: 'auto', height: height+'px'}" id="mapArea"></div>
  </div>
</template>
<script>
import json from './xianning.json'
import MapMonitor from './MapMonitor'

export default {
  components: { MapMonitor },
  data() {
    return {
      height: 400,
      option: {
        title: {
          text: '咸宁市行政区划图'
        },
        geo: {
          map: 'XN',
          label: {
            show: true,
            color: 'rgba(0,0,0,0.8)',
            fontSize: '16',
            position: 'right'
          },
          itemStyle: {
            areaColor: 'rgba(44,84,154,1)',
            borderColor: '#fff'
          },
          emphasis: {
            itemStyle: {
              areaColor: 'rgba(44,84,154,0)',
              borderColor: 'blue'
            }
          },
          select: {
            itemStyle: {
              areaColor: 'rgba(44,84,154,1)',
              borderColor: '#fff'
            }
          },
          regions: [{
            name: '嘉鱼县',
            itemStyle: { areaColor: 'rgb(250,200,135)' }
          }, {
            name: '咸安区',
            itemStyle: { areaColor: 'rgb(254,170,254)' }
          }, {
            name: '赤壁市',
            itemStyle: { areaColor: 'rgb(227,197,0)' }
          }, {
            name: '通山县',
            itemStyle: { areaColor: 'rgb(194,254,162)' }
          }, {
            name: '崇阳县',
            itemStyle: { areaColor: 'rgb(165,243,253)' }
          }, {
            name: '通城县',
            itemStyle: { areaColor: 'rgb(252,190,192)' }
          }]
        }
      }
    }
  },
  mounted() {
    this.loadEChart()
  },
  methods: {
    //初始化echarts
    loadEChart() {
      let echarts = require('echarts')
      echarts.registerMap('XN', json)

      let myChart = echarts.init(document.getElementById('mapArea'))
      myChart.setOption(this.option)

      this.height = document.body.clientHeight - 140
      myChart.getDom().style.height = this.height + 'px'
      myChart.resize()
      window.onresize = myChart.resize

      let _this = this;
      myChart.on('click', function(params) {
        console.log('click map:', params.name, params);
        // _this.$router.push({name:"mapMonitor",params:{cityName:params.name}});
        _this.$router.push({path:'/gis/monitor',query: {city:params.name}});
      });
    }
  }
}
</script>