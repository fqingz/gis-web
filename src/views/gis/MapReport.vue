<template>
  <a-modal title="线索报告"
         :visible="visible" @ok="handleOk()" @cancel="handleCancel()">
    <div>地点：{{markerInfo.address}}</div>
    <div>经纬度：北纬{{markerInfo.lat}} 东经{{markerInfo.lng}}</div>
    <div>告警类型：{{markerInfo.type}}</div>
    <div>实景地图：</div>
    <img alt="example" style="width: 100%"/>
  </a-modal>
</template>
<script>
export default {
  name: 'MapReport',
  data() {
    return {
      visible: false,
      markerInfo: {
        lng: '',
        lat: '',
        address: '',
        type: '',
        level: '',
        time: ''
      }
    }
  },
  methods: {
    showModal(markerInfo) {
      this.markerInfo = markerInfo;
      this.visible = true;
    },
    handleOk() {
      this.visible = false;
    },
    handleCancel() {
      this.visible = false;
    }
  }
}
</script>
<style>

</style>