<template>
  <div>
    <div :style="{width: 'auto', height: height+'px'}" id="mapArea"></div>
  </div>
</template>
<script>
import json from './xianning.json'
import MapMonitor from './MapMonitor'
import store from '@/store/'
import { getAction, getRoleList } from '../../api/manage'
import { queryUserRole } from '../../api/api'

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
          select: {
            itemStyle: {
              areaColor: 'rgba(44,84,154,1)',
              borderColor: '#fff'
            }
          },
          regions: [{
            name: '嘉鱼县',
            itemStyle: { areaColor: 'rgb(250,200,135)' },
          }, {
            name: '咸安区',
            itemStyle: { areaColor: 'rgb(254,170,254)' },
          }, {
            name: '赤壁市',
            itemStyle: { areaColor: 'rgb(227,197,0)' },
          }, {
            name: '通山县',
            itemStyle: { areaColor: 'rgb(194,254,162)' },
          }, {
            name: '崇阳县',
            itemStyle: { areaColor: 'rgb(165,243,253)' },
          }, {
            name: '通城县',
            itemStyle: { areaColor: 'rgb(252,190,192)' },
          }]
        },
        orgCode: '',
        currDepartName: ''
      }
    }
  },
  mounted() {
    this.loadEChart()
  },
  methods: {
    //初始化echarts
    loadEChart() {
      let _this = this;
      let echarts = require('echarts')
      echarts.registerMap('XN', json)

      let myChart = echarts.init(document.getElementById('mapArea'))
      myChart.setOption(this.option)

      this.height = document.body.clientHeight - 140
      myChart.getDom().style.height = this.height + 'px'
      myChart.resize()
      window.onresize = myChart.resize

      let url = "/sys/user/getCurrentUserDeparts"
      _this.currDepartName = '';
      getAction(url).then(res=>{
        if(res.success){
          console.log(res);
          let departs = res.result.list
          let orgCode = res.result.orgCode
          if(departs && departs.length>0){
            for(let i of departs){
              if(i.orgCode === orgCode){
                _this.currDepartName = i.departName

                if(_this.currDepartName !== '咸宁市') {
                  _this.option.geo.emphasis={}
                  for(let region of _this.option.geo.regions) {
                    if(region.name === _this.currDepartName) {
                      region.emphasis= {
                        itemStyle: {
                          areaColor: 'rgba(44,84,154,0)',
                            borderColor: 'blue'
                        }
                      }
                    }else {
                      region.emphasis= {
                        itemStyle: region.itemStyle
                      }
                    }
                  }
                } else {
                  _this.option.geo.emphasis= {
                    itemStyle: {
                      areaColor: 'rgba(44,84,154,0)',
                        borderColor: 'blue'
                    }
                  }
                }
                myChart.setOption(_this.option)

                myChart.on('click', function(params) {
                  // 验证权限
                  if(_this.currDepartName === '咸宁市' || _this.currDepartName === params.name) {
                    _this.$router.push({path:'/gis/monitor',query: {city:params.name}});
                  }
                });
                console.log(_this.option)
                break
              }
            }
          }
          // this.departSelected = orgCode
          // this.departList  = departs
          // if(this.currDepartName){
          //   this.currTitle ="部门切换（当前部门 : "+this.currDepartName+"）"
          // }

        }
      })


    },
  }
}
</script>