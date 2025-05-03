<template>
  <div style="min-height: 1000px; background-color: #f6f6f8">
<!--    <img src="@/assets/imgs/banner.jpg" alt="" style="width: 100%; height: 120px">-->
    <div style="margin: 25px auto; width: 70%; text-align: center">
      <el-input size="large" v-model="data.name" placeholder="请输入你感兴趣的职位" style="width: 500px; margin-right: 5px"></el-input>
      <el-button size="large" type="info" @click="search">搜索</el-button>
    </div>
    <div style="margin: 0 auto; width: 70%; text-align: center">
      <div>
        <el-tabs v-model="data.activeName" class="demo-tabs" @tab-change="handleClick">
          <el-tab-pane v-for="item in data.industryData" :label="item.name" :name="item.id">
            <el-row :gutter="10">
             <el-col :span="8" v-for="it in data.positionData" style="margin-bottom: 20px">
               <div class="card" style="cursor: pointer" @click="navTo('/front/positionDetail?id=' + it.id)">
                 <div style="display: flex; padding: 0 5px">
                   <div style="flex: 1; text-align: left; font-size: 16px">{{ it.name }}</div>
                   <div style="width: 100px; text-align: right; color: red">{{ it.salary }}</div>
                 </div>
                 <div style="margin: 10px 0; padding: 0 5px; text-align: left">
                   <el-tag style="margin-right: 5px" type="info" v-for="tag in it.tagList">{{ tag }}</el-tag>
                 </div>
                 <div style="display: flex; align-items: center; padding: 10px 5px">
                   <div style="width: 35px"><img :src="it.employAvatar" alt="" style="width: 35px; height: 35px; border-radius: 5px; border: 1px solid #cccccc"></div>
                   <div style="width: 80px">{{ it.employName }}</div>
                   <div style="flex: 1">{{ item.name }}</div>
                   <div style="width: 80px">{{ it.employStage }}</div>
                 </div>
               </div>
             </el-col>
            </el-row>
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>


      <div style="min-height: 1000px; background-color: #f6f6f8">
        <!-- 搜索框 -->
        <div style="margin: 25px auto; width: 70%; text-align: center">
          <el-input v-model="searchKey" placeholder="请输入职位关键词" style="width: 500px"></el-input>
          <el-button type="primary" @click="handleSearch">搜索</el-button>
        </div>
        <div style="margin: 30px auto; width: 70%; display: flex; gap: 20px">
          <div style="margin-top: 10px; height: 350px;width: 530px;margin-left: auto; " class="card" id="pie1"></div>
          <div style="margin-top: 10px; height: 350px;width: 525px;margin-right: auto; " class="card" id="bar1"></div>
        </div>

        <!-- 学历-经验双图表 -->
        <div style="margin: 30px auto; width: 70%; display: flex; gap: 20px">
          <div ref="eduChartRef" style="flex: 1; height: 400px; background: white; border-radius: 8px; padding: 20px"></div>
          <div ref="expChartRef" style="flex: 1; height: 400px; background: white; border-radius: 8px; padding: 20px"></div>
        </div>

        <!-- 城市需求横向柱状图 -->
        <div style="margin: 30px auto; width: 70%" class="card">
          <div ref="cityDemandChartRef" style="height: 500px; background: white; border-radius: 8px; padding: 20px"></div>
        </div>

        <!-- 岗位数量统计 -->
        <div style="margin: 30px auto; width: 70%">
          <div ref="jobCountChartRef" style="height: 500px; background: white; border-radius: 8px; padding: 20px"></div>
        </div>

        <!-- 薪资-地址双图表 -->
        <div style="margin: 30px auto; width: 70%; display: flex; gap: 20px">
          <div ref="salaryChartRef" style="flex: 1; height: 500px; background: white; border-radius: 8px; padding: 20px"></div>
          <div ref="addressChartRef" style="flex: 1; height: 500px; background: white; border-radius: 8px; padding: 20px"></div>
        </div>
      </div>
  </div>
</template>

<script setup>
import {reactive} from "vue";
import request from "@/utils/request.js";
import {ElMessage} from "element-plus";

import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as echarts from 'echarts'

// 图表实例
const eduChartRef = ref(null)
const expChartRef = ref(null)
const cityDemandChartRef = ref(null)
const jobCountChartRef = ref(null)
const salaryChartRef = ref(null)
const addressChartRef = ref(null)

let eduChart = null
let expChart = null
let cityDemandChart = null
let jobCountChart = null
let salaryChart = null
let addressChart = null


// 学历数据
const eduData = {
  categories: ['不限', '中专/高中', '大专', '本科', '硕士', '博士'],
  values: [19, 52, 75, 192, 258, 27],
  colors: ['#5470c6', '#91cc75', '#fac858', '#ee6666', '#73c0de', '#3ba272']
}

// 经验数据
const expData = {
  categories: ['不限', '1年以下', '1-3年', '3-5年', '5-10年', '10年以上'],
  values: [14, 30, 303, 240, 16, 5],
  color: '#ee6666'
}

// 初始化学历柱状图
const initEduChart = () => {
  eduChart = echarts.init(eduChartRef.value)
  eduChart.setOption({
    title: { text: '学历要求统计', left: 'center' },
    tooltip: { trigger: 'axis' },
    xAxis: { type: 'value' },
    yAxis: {
      type: 'category',
      data: eduData.categories,
      axisLabel: { interval: 0 }
    },
    series: [{
      type: 'bar',
      data: eduData.values,
      itemStyle: {
        color: params => eduData.colors[params.dataIndex]
      }
    }]
  })
}

// 初始化经验折线图
const initExpChart = () => {
  expChart = echarts.init(expChartRef.value)
  expChart.setOption({
    title: { text: '工作年限统计', left: 'center' },
    tooltip: { trigger: 'axis' },
    xAxis: {
      type: 'category',
      data: expData.categories,
      axisLabel: { rotate: 30 }
    },
    yAxis: { type: 'value' },
    series: [{
      type: 'line',
      data: expData.values,
      smooth: true,
      itemStyle: { color: expData.color }
    }]
  })
}

// 初始化城市需求横向柱状图
const initCityDemandChart = () => {
  cityDemandChart = echarts.init(cityDemandChartRef.value)
  cityDemandChart.setOption({
    title: { text: '城市需求统计', left: 'center' },
    tooltip: { trigger: 'axis' },
    xAxis: { type: 'value' },
    yAxis: {
      type: 'category',
      data: ['北京','上海','深圳','广州','杭州','成都','武汉','南京','厦门','苏州'],
      inverse: true
    },
    series: [{
      type: 'bar',
      data: [14582, 12547, 11527, 10020, 9005, 7500, 5200, 2200, 2050, 1800],
      itemStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 1, 0, [
          { offset: 0, color: '#83bff6' },
          { offset: 0.5, color: '#188df0' },
          { offset: 1, color: '#08519c' }
        ])
      }
    }]
  })
}

// 初始化岗位数量统计
const initJobCountChart = () => {
  jobCountChart = echarts.init(jobCountChartRef.value)
  jobCountChart.setOption({
    title: { text: '工作岗位数量统计', left: 'center' },
    tooltip: { trigger: 'axis' },
    xAxis: {
      type: 'category',
      data: ['C++开发','Android开发','iOS开发','运维工程师','产品经理','前端开发','后端开发','Java开发','PHP开发','产品销售','嵌入式开发','IU开发'],
      axisLabel: { rotate: 45 }
    },
    yAxis: { type: 'value' },
    series: [{
      type: 'bar',
      data: [8000, 12000, 15000, 18000, 21000, 24000, 18000, 15000, 12000, 9000, 6000, 3000],
      itemStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: '#6a8df0' },
          { offset: 0.7, color: '#3a5dc7' },
          { offset: 1, color: '#1a3d9e' }
        ])
      }
    }]
  })
}

// 初始化薪资折线图
const initSalaryChart = () => {
  salaryChart = echarts.init(salaryChartRef.value)
  salaryChart.setOption({
    title: { text: '薪资幅度趋势', left: 'center' },
    tooltip: { trigger: 'axis' },
    xAxis: {
      type: 'category',
      data: ['在校/应届','经验不限','1年以内','1-3年','3-5年','5-10年','10年以上'],
      axisLabel: { rotate: 30 }
    },
    yAxis: { type: 'value' },
    series: [{
      type: 'line',
      data: [1000, 2000, 4000, 3000, 5000, 7000, 6000],
      smooth: false                                                                                                                                                                                             ,
      itemStyle: { color: '#ee6666' }
    }]
  })
}

// 初始化地址环状图
const initAddressChart = () => {
  addressChart = echarts.init(addressChartRef.value)
  addressChart.setOption({
    title: { text: '公司地址分布', left: 'center' },
    tooltip: { trigger: 'item' },
    series: [{
      type: 'pie',
      radius: ['40%', '70%'],
      data: [
        { value: 1048, name: '北京' },
        { value: 735, name: '上海' },
        { value: 580, name: '深圳' },
        { value: 484, name: '广州' },
        { value: 300, name: '其他' }
      ],
      itemStyle: {
        borderRadius: 10,
        borderColor: '#fff',
        borderWidth: 2
      }
    }]
  })
}

// 统一初始化
const initAllCharts = () => {
  initEduChart()
  initExpChart()
  initCityDemandChart()
  initJobCountChart()
  initSalaryChart()
  initAddressChart()
}

// 响应式处理
const handleResize = () => {
  ;[eduChart, expChart, cityDemandChart, jobCountChart, salaryChart, addressChart].forEach(chart => {
    chart?.resize()
  })
}

// 生命周期


onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  ;[eduChart, expChart, cityDemandChart, jobCountChart, salaryChart, addressChart].forEach(chart => {
    chart?.dispose()
  })
})
//原来内容
const data = reactive({
  name: null,
  advertiseData: [],
  centerAd: {},
  leftAd: {},
  rightAd: {},
  centerDownAd: {},
  leftDownAd: {},
  rightDownAd: {},
  activeName: null,
  industryData: [],
  positionData: []
})

const loadAdvertise = () => {
  request.get('/advertise/selectAll').then(res => {
    if (res.code === '200') {
      data.advertiseData = res.data
      // 做一下过滤，把六个广告位信息过滤出来
      let centerArr = res.data.filter(v => v.location === '中心大图')
      data.centerAd = centerArr && centerArr.length > 0 ? centerArr[0] : {}
      let leftArr = res.data.filter(v => v.location === '左侧大图')
      data.leftAd = leftArr && leftArr.length > 0 ? leftArr[0] : {}
      let rightArr = res.data.filter(v => v.location === '右侧大图')
      data.rightAd = rightArr && rightArr.length > 0 ? rightArr[0] : {}

      let centerDownArr = res.data.filter(v => v.location === '中心小图')
      data.centerDownAd = centerDownArr && centerDownArr.length > 0 ? centerDownArr[0] : {}
      let leftDownArr = res.data.filter(v => v.location === '左侧小图')
      data.leftDownAd = leftDownArr && leftDownArr.length > 0 ? leftDownArr[0] : {}
      let rightDownArr = res.data.filter(v => v.location === '右侧小图')
      data.rightDownAd = rightDownArr && rightDownArr.length > 0 ? rightDownArr[0] : {}
    } else {
      ElMessage.error(res.msg)
    }
  })
}

const loadIndustry = () => {
  request.get('/industry/selectAll').then(res => {
    if (res.code === '200') {
      data.industryData = res.data
      data.activeName = data.industryData[0].id
      handleClick(data.activeName)
    } else {
      ElMessage.error(res.msg)
    }
  })
}

const handleClick = (industryId) => {
  request.get('/position/selectAll', {
    params: {
      industryId: industryId,
      status: '审核通过'
    }
  }).then(res => {
    if (res.code === '200') {
      data.positionData = res.data
    } else {
      ElMessage.error(res.msg)
    }
  })
}
const navTo = (url) => {
  location.href = url
}
const search = () => {
  location.href = '/front/search?name=' + data.name
}

loadAdvertise()
loadIndustry()





const loadPie1Data = () =>{

}
onMounted(() => {
  loadPie1Data()





  initAllCharts()
  window.addEventListener('resize', handleResize)
})

</script>
