import JsPDF from "jspdf"
<template>
  <a-modal title="导出报告" class="mapReport" :width="1200"
           :visible="visible" @ok="handleOk()" @cancel="handleCancel()" :footer="null">

    <div id="pdfDom">
      <h2 style="text-align: center;">{{markerInfo.city}}{{reportTitle}}</h2>

      <div><span class="label">经纬度：</span>北纬{{markerInfo.lat}} 东经{{markerInfo.lng}}</div>
      <div><span class="label">告警类型：</span>{{markerInfo.type}}</div>
      <div><span class="label">实景地图：</span></div>
      <div style="height: 400px;">
        <div class="mapArea">
          <br style="clear:both"/>
          <img :src="leftImgUrl" class="mapImg">
          <div class="center">{{markerInfo.date1}}</div>
        </div>
        <div class="mapArea">
          <br style="clear:both"/>
          <img :src="rightImgUrl" class="mapImg">
          <div class="center">{{markerInfo.date2}}</div>
        </div>
      </div>

      <div style="height: 80px;">
        <div style="float: right; min-width:300px;">
          <div><span class="label">制作时间：</span>
            <a-input class="input" border="false" v-model="createDate"></a-input>
          </div>
          <div><span class="label">制作地点：</span>
            <a-input class="input" v-model="createAddr"></a-input></div>
          <div><span class="label">制作人：</span>
            <a-input class="input" v-model="createUser"></a-input></div>
        </div>
      </div>
    </div>

    <div style="text-align: center">
      <a-button @click="handleReport" style="padding: 0 20px;">导 出</a-button>
    </div>
  </a-modal>
</template>
<script>
import { getFileAccessHttpUrl } from '../../api/manage'
import html2Canvas from 'html2canvas'
import JsPDF from 'jspdf'
import moment from 'moment'

export default {
  name: 'MapReport',
  data() {
    return {
      visible: false,
      markerInfo: {
        city: '',
        date1: '',
        date2: ''
      },
      leftImgUrl: '',
      rightImgUrl: '',
      createDate: '',
      createAddr: '',
      createUser: '',
      reportTitle: '人民检察院公益诉讼案件线索报告',
      htmlTitle: ''
    }
  },
  methods: {
    showModal(markerInfo) {
      this.markerInfo = markerInfo;
      console.log(markerInfo.path1);
      console.log(markerInfo.path2);
      this.leftImgUrl = getFileAccessHttpUrl(markerInfo.path1)
      this.rightImgUrl = getFileAccessHttpUrl(markerInfo.path2)
      this.createDate = moment(new Date()).format('YYYY-MM-DD HH:mm:ss');
      this.visible = true
    },
    handleOk() {
      this.visible = false
    },
    handleCancel() {
      this.visible = false
    },
    handleReport() {
      let title = this.markerInfo.city + this.reportTitle + "_" + this.createDate;

      html2Canvas(document.querySelector('#pdfDom'), {
        useCORS: true
      }).then(function(canvas) {
          let contentWidth = canvas.width
          let contentHeight = canvas.height
          let imgWidth = 820
          let imgHeight = 820 / contentWidth * contentHeight
          let pageData = canvas.toDataURL('image/jpeg', 1.0)
          let PDF = new JsPDF('l', 'pt', 'a4')
          PDF.addImage(pageData, 'JPEG', 15, 45, imgWidth, imgHeight)
          PDF.save(title + '.pdf')
        }
      )
    }
  }
}
</script>
<style scoped lang="less">
  .mapReport {
    .mapArea {
      width: 550px;
      height: 350px;
      margin: 0 10px;
      float: left;
      overflow: hidden;
    }

    .mapImg {
      border: 1px dashed blue;
      max-height: 300px;
      max-width: 550px;
    }

    .label {
      display: inline-block;
      margin-right: 10px;
    }

    .center {
      text-align: center;
    }

    .input {
      width: 200px;
      border-top-width: 0;
      border-right-width: 0;
      border-left-width: 0;
      height: 20px;
      line-height: 1.5;
      padding: 0 5px;
    }
  }
</style>