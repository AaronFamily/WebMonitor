<template>
  <div class="container">
    <div class="stable" >
      <pie-chart class="arrange-v" ref="userOnline"></pie-chart>
      <bar-chart class="arrange-v" ref="interfaceTime"></bar-chart>
      <bar-chart class="arrange-v" ref="loadPageTime"></bar-chart>
    </div>
    <div class="change">
      <map-chart ref="map"></map-chart>
      <div class="mapDetail" v-show="showMapDetail">
        {{mapDetail}}
        <div class="close" v-on:click="showMapDetail=false"></div>
      </div>
      <div class="systemTitle" style="background: url(static/title.png);background-size: 440px 100px;"></div>
      <div class="jumpToDetail" style="background-image: url(static/jump.jpg);" title="跳转至详情页面" v-on:click="jumpToDetail"></div>
    </div>
    <div class="stable">
      <pie-chart class="arrange-v" ref="browser"></pie-chart>
      <bar-chart class="arrange-v" ref="interfaceError"></bar-chart>
      <div class="arrange-v" style="width:300px;height:30%">
        <ul>
          <li><span>近一月接口错误总数：<span class="resultSpan">{{errorTotal}}</span></span></li>
          <li><span>近一月接口平均耗时： <span class="resultSpan">{{averageTime}}</span></span></li>
          <li style="position:relative">
            <span>近一月异常人员数量：<span class="resultSpan" v-on:mouseover="showUserId=true" v-on:mouseleave="showUserId=false" style="text-decoration: underline;cursor: pointer;">{{errorUser}}</span></span>
            <div v-show="showUserId" class="errorTip"><span>错误用户ID：{{errorUserID}}</span></div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import pieChart from './pieChart.vue';
  import barChart from './barChart.vue';
  import mapChart from './mapChart.vue';
  import { mapShow, userOnline, interfaceTime, loadPageTime, browser, interfaceError, staticList, detailList, queryErrorByLocation } from '../dataService/api';
  import { appUtil } from '../config';
  import { Utils } from '../common/js/utils.js';
  var self;
  export default {
    name: 'Main',
    data () {
      return {
        showUserId:false,
        errorTotal:0,
        averageTime:'无数据',
        errorUser:'无数据',
        errorUserID: '无数据',
        mapDetail: '',
        showMapDetail:false
      }
    },
    mounted:function(){
      
      this.$nextTick(()=>{
        self = this;

        initUserOnline();
        getLoadPageTime();
        getInterfaceTime();
        getBrowser();
        getInterfaceError();
        getStaticList();
        getMapShow();
      })
    },
    components: {
      pieChart,
      barChart,
      mapChart
    },
    methods: {
      jumpToDetail: jumpToDetail
    }
  }

  function initUserOnline(){
    userOnline().then(function(res){
      self.$refs.userOnline.init({
        content: {
          id: 'userOnlinePie',
          height:'30%',
          width: '300px',
          ref: 'userOnline'
        },
        option: {
          title: '用户在线、离线数统计',
          data: res.data
        }
      });
    })
  }

  function getInterfaceTime(){
    interfaceTime().then(function(res){

      var param = {
        content:{
          id: 'interfaceTimePie',
          height: '30%',
          width: '300px',
          ref: 'interfaceTime'
        },
        option: {
          title: '接口请求耗时TOP10省份',
          xAxis: [],
          data: []
        }
      }
      if(!res.errcode){
        res.data.forEach(function(ele){
          param.option.xAxis.push(ele.name);
          param.option.data.push(ele.value);
        })
      }

      self.$refs.interfaceTime.init(param);
    })
  }

  function getLoadPageTime(){
    loadPageTime().then(function(res){

      var param = {
        content:{
          id: 'loadPageTimePie',
          height: '30%',
          width: '300px',
          ref: 'loadPageTime'
        },
        option: {
          title: '页面渲染耗时TOP10省份',
          xAxis: [],
          data: []
        }
      }
      if(!res.errcode){
        res.data.forEach(function(ele){
          param.option.xAxis.push(ele.name);
          param.option.data.push(ele.value);
        })
      }

      self.$refs.loadPageTime.init(param);
    })
  }

  function getBrowser(){
    browser().then(function(res){
      res.data.forEach(function(ele){
        ele.name = ele.name.substring(ele.name.indexOf('/') + 1,ele.name.length)
      });
      self.$refs.browser.init({
        content: {
          id: 'browserPie',
          height:'30%',
          width: '300px',
          ref: 'browser'
        },
        option: {
          title: 'Chrome各版本使用占比',
          data: res.data
        }
      });
    })
  }

  function getInterfaceError(){
    interfaceError().then(function(res){
      var param = {
        content:{
          id: 'interfaceErrorPie',
          height: '30%',
          width: '300px',
          ref: 'interfaceError'
        },
        option: {
          title: '接口错误统计TOP10省份',
          xAxis: [],
          data: []
        }
      }
      if(!res.errcode){
        res.data.forEach(function(ele){
          param.option.xAxis.push(ele.name);
          param.option.data.push(ele.value);
        })
      }

      self.$refs.interfaceError.init(param);
    })
  }

  function getStaticList(){
    staticList().then(function(res){
      if(!res.errcode){
        self.errorTotal = res.data.interfaceError || 0 + ' 个';
        self.averageTime = res.data.loadTime || 0 + ' 毫秒';
        self.errorUser = res.data.errUserCount || 0 + ' 个';
        self.errorUserID = res.data.erruserName || '无错误用户';
      }
    })
  }

  function getMapShow(){
    self.$refs.map.init({
      content:{
        height: '100%',
        width: '100%',
        ref: 'map',
        id:'mainMap'
      },
      option:{}
    })
    mapShow().then(function(res){
      if(!res.errcode){
        var mapData = [];
        res.data.forEach(function(ele){
          mapData.push({
            name:'',
            value: ele.users,
            coord: ele.geometry.split(',')
          })
        })
        self.$refs.map.setData(mapData)
      }
    })

    self.$refs.map.mapClick = function(data){
      var param = {
        location: data.data.coord.join(',')
      }
      queryErrorByLocation({parameter:JSON.stringify(param)}).then(function(res){
        if(!res.errcode){
          self.showMapDetail= true;
          if(res.data.length != 0){
            var tempName = [];
            res.data.forEach(function(ele){
              tempName.push(ele.userName);
            })
            self.mapDetail = '接口报错用户有： ' + tempName.join(', ');
          }else{
            self.mapDetail = '无报错信息';   
          }
        }
      })
    }
  }

  function jumpToDetail(){
    window.open('index.html#/query');
  }

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
  .container {
    height: 100%;
    width: 100%;
  }
  .container{
    height: 100%;
    display:flex; /*设为伸缩容器*/
    flex-flow:row; /*伸缩项目单行排列*/
    align-items: center; /* 上下居中 */
    .stable {
      width:350px; /*固定宽度*/
      height: 100%;
      text-align: center;
      .arrange-v{
        height:30%;
        width: 300px;
        margin: 15px auto;
        padding-top: 10px;
        box-shadow: 0px 0px 4px #009683;
      }
      ul {
        list-style: none;
        text-align: left;
        li{
          margin: 10px 0px 10px -18px;
          color: #fff;
          font-size: 13px;
          line-height: 23px;
          .resultSpan{
            color: #55decd;
            margin-left: 5px;
            font-size: 15px; 
          }
        }
        .errorTip{
          position: absolute;
          left: 149px;
          top: 30px;
          padding: 5px 10px;
          background-color: #5b7b8a;
          border-radius: 5px;
          box-shadow: rgb(255, 255, 255) 0px 0px 7px;
        }
      }
    }
    .change {
      flex:1; /*这里设置为占比1，填充满剩余空间*/
      height: 100%;
      min-width: 600px;
      position: relative;
      .mapDetail{
        width: 200px;
        min-height: 80px;
        position: absolute;
        top: 100px;
        background-color: #999;
        left: 30px;
        -webkit-box-shadow: 0 0 6px #fff;
        box-shadow: 0 0 6px #fff;
        color: #fff;
        padding: 10px 0 0 10px;
        font-size: 14px;
      }
      .systemTitle{
        position: absolute;
        width: 440px;
        height: 100px;
        top: 36px;
        background-size: 440px 100px;
        right: 0;
        margin: auto;
        left: 0;
      }
      .jumpToDetail{
        position: absolute;
        width: 30px;
        height: 30px;
        background-size: 100%;
        top: 60px;
        border-radius: 4px;
        right: 52px;
        cursor: pointer;
      }
    }

    .close {
        position: absolute;
        right: -14px;
        top: -11px;
        width: 20px;
        height: 20px;
        background: silver;
        border-radius: 25px;
        -webkit-box-shadow: 2px 2px 5px 0px black;
        box-shadow: 2px 2px 5px 0px black;
        cursor: pointer;
    }

    .close:hover {
        background: #ddd;
    }
    .close:before {
        position: absolute;
        content: '';
        width: 17px;
        height: 2px;
        background: white;
        -webkit-transform: rotate(45deg);
        transform: rotate(45deg);
        top: 9px;
        left: 2px;
    }
    .close:after{
        content: '';
        position: absolute;
        width: 17px;
        height: 2px;
        background: white;
        -webkit-transform: rotate(-45deg);
        transform: rotate(-45deg);
        top: 9px;
        left: 2px;
    }
  }
</style>

