<template>
  <div>
    <div id="container" :style="{height: height+'px'}"></div>

    <map-report ref="mapReport"></map-report>
  </div>
</template>
<script>

import { BMPGL } from './bmpgl'
import MapReport from './MapReport'
import { getAction } from '../../api/manage'

export default {
  name: 'MapMonitor',
  components: {MapReport},
  data() {
    return {
      height: 400,
      ak: 'MHGAkE6LDRRKvT74Xt4wIA6TxgmDePHX',
      center: '咸宁市',

      currentMarkerInfo: {},
      url: {
        query: "/gis/gisImage/queryImg",
      }
    }
  },
  created() {
    this.center = this.$route.query.city
    if ('咸宁市,嘉鱼县,赤壁市,咸安区,通山县,崇阳县,通城县'.split(',').includes(this.center)) {
      console.log('city:', this.center)
      this.initMap()
      this.height = document.body.clientHeight - 140
    }
  },
  methods: {
    show(cityName) {
      this.center = cityName
      this.visible = true
      this.initMap()
      this.height = document.body.clientHeight - 140
    },

    initMap() {
      //传入密钥获取地图回调
      BMPGL(this.ak)
        .then(BMapGL => {
          // 创建地图实例
          let map = new BMapGL.Map('container')
          // 初始化地图，设置中心点坐标和地图级别
          // new BMapGL.Point(118.477, 31.346)
          map.centerAndZoom(this.center, 11)
          map.enableScrollWheelZoom(true)//开启鼠标滚轮缩放
          map.addControl(new BMapGL.ScaleControl())// 添加比例尺控件
          map.addControl(new BMapGL.ZoomControl())// 添加比例尺控件

          // map.setMinZoom(9);
          // map.setMaxZoom(15);
          // map.setMapStyleV2({ styleJson: custom_map });
          // map.setDisplayOptions({
          //   skyColors: ["rgba(0, 0, 0, 0)"]
          // });

          // 设置行政区域
          let bd = new BMapGL.Boundary()
          bd.get(this.center, function(rs) {
            // console.log('外轮廓：', rs.boundaries[0]);
            // console.log('内镂空：', rs.boundaries[1]);
            let hole = new BMapGL.Polygon(rs.boundaries, {
              fillColor: 'blue',
              fillOpacity: 0.2
            })
            map.addOverlay(hole)
          })

          // 增加返回按钮
          this.addBackControl(map, this)

          map.addEventListener('click', function(e) {
            console.log('位置：' + e.latlng.lng + ',' + e.latlng.lat)
          })

          let markerInfo = {
            lng: 113.72,
            lat: 29.71,
            city: '赤壁市',
            type: '森林砍伐',
            time: '2021-04-30 10:05:20',
            // path1: 'temp/1_1620832397420.jpg',
            // path2: 'temp/1_1620832397420.jpg',
            // date1: '2021-04-10 11:00:44',
            // date2: '2021-05-10 11:00:44',
          }

          let _this = this

          getAction(this.url.query, {}).then((res) => {
            if (res.success && res.result) {
              if(res.result.length === 2){
                markerInfo.path1 = res.result[0].image;
                markerInfo.path2 = res.result[1].image;
                markerInfo.date1 = res.result[0].createTime;
                markerInfo.date2 = res.result[1].createTime;

                // 创建点标记
                let point = new BMapGL.Point(markerInfo.lng, markerInfo.lat)
                let marker = new BMapGL.Marker(point)
                // 在地图上添加点标记
                map.addOverlay(marker)

                let markerInfoStr = JSON.stringify(markerInfo)
                console.log('markerInfoStr', markerInfoStr)

                let content = `<ul style="margin-left:-20px;line-height:14px;font-size:12px;">
                   <li>破坏类型：${markerInfo.type}</li>
                   <li>监测时间：${markerInfo.time}</li>
                   <a class="markerBtn" markerInfo='${markerInfoStr}'>导出报告</a>
                </ul>`
                let infoWindow = new BMapGL.InfoWindow(content, { title: markerInfo.address })  // 创建信息窗口对象


                marker.addEventListener('click', function() {
                  map.openInfoWindow(infoWindow, point) //开启信息窗口

                  let infoWindows = document.getElementsByClassName('markerBtn')
                  for (let infoWindow of infoWindows) {
                    infoWindow.onclick = function() {
                      _this.handlerDetail(JSON.parse(infoWindow.getAttribute('markerInfo')), _this)
                    }
                  }
                })

              }
            }
          })


        })
    },

    handlerDetail(markerInfo, _this) {
      _this.$refs.mapReport.showModal(markerInfo);
    },

    addBackControl(map, _this) {
      //定义一个控件类
      function BackControl() {
        this.defaultAnchor = BMAP_ANCHOR_TOP_LEFT
        this.defaultOffset = new BMapGL.Size(20, 20)
      }

      //通过JavaScript的prototype属性继承于BMap.Control
      BackControl.prototype = new BMapGL.Control()

      //自定义控件必须实现自己的initialize方法，并且将控件的DOM元素返回
      //在本方法中创建个div元素作为控件的容器，并将其添加到地图容器中
      BackControl.prototype.initialize = function(map) {
        //创建一个dom元素
        let div = document.createElement('div')
        //添加文字说明
        div.appendChild(document.createTextNode('返 回'))
        // 设置样式
        div.style.cursor = 'pointer'
        div.style.padding = '6px 15px'
        div.style.boxShadow = '0 2px 6px 0 rgba(27, 142, 236, 0.5)'
        div.style.borderRadius = '4px'
        div.style.backgroundColor = 'white'
        // 绑定事件,点击一次放大两级
        div.onclick = function(e) {
          _this.$router.go(-1)
        }
        // 添加DOM元素到地图中
        map.getContainer().appendChild(div)
        // 将DOM元素返回
        return div
      }
      //添加到地图中
      map.addControl(new BackControl())
    }
  }
}
</script>

<style lang="less" scoped>
  .map {
    width: 100%;
    height: 400px;
  }
</style>