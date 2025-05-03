<template>
  <div style="padding: 30px 0; background-color: #f6f6f8">
    <div style="width: 55%; padding: 30px; background-color: white; margin: 0 auto; border-radius: 10px">
      <div style="font-size: 22px; text-align: center; font-weight: bold">{{ data.resumeData.name }}</div>
      <div style="margin-top: 40px; display: flex">
        <img :src="data.resumeData.userAvatar" alt="" style="width: 150px; height: 150px; border-radius: 10px">
        <div style="margin-left: 50px; flex: 1">
          <div style="font-size: 20px">{{ data.resumeData.username }}</div>
          <div style="margin-top: 10px; font-size: 15px">
            <span>{{ data.resumeData.sex }}</span>
            <span style="margin: 0 20px">|</span>
            <span>{{ data.resumeData.education }}</span>
            <span style="margin: 0 20px">|</span>
            <span>{{ data.resumeData.experience }}</span>
          </div>
          <div style="margin-top: 10px; font-size: 15px">电话：{{ data.resumeData.phone }}</div>
          <div style="margin-top: 10px; font-size: 15px">邮箱：{{ data.resumeData.email }}</div>
          <div style="margin-top: 10px; font-size: 15px">期望薪资：{{ data.resumeData.salary }}</div>
        </div>
      </div>
      <div style="margin-top: 20px; font-size: 16px" v-if="data.resumeData.eduExpList.length">
        <div style="font-weight: bold; padding: 10px; background: linear-gradient(to right, #8edfee, white)">教育经历：</div>
        <div style="margin-top: 10px; margin-bottom: 20px" v-for="item in data.resumeData.eduExpList">
          <div style="display: flex; align-items: center">
            <div style="flex: 1; color: #00bebd">{{ item.start }} ~ {{ item.end ? item.end : '至今' }}</div>
            <div style="flex: 1; color: #00bebd; font-weight: bold">{{ item.school }}</div>
            <div style="flex: 1; color: #00bebd; font-weight: bold">{{ item.speciality }}（{{ item.education }}）</div>
          </div>
          <div style="margin-top: 10px">主修课程：{{ item.course }}</div>
        </div>
        <div></div>
      </div>
      <div style="margin-top: 20px; font-size: 16px" v-if="data.resumeData.workExpList.length">
        <div style="font-weight: bold; padding: 10px; background: linear-gradient(to right, #8edfee, white)">工作经历：</div>
        <div style="margin-top: 10px; margin-bottom: 20px" v-for="item in data.resumeData.workExpList">
          <div style="display: flex; align-items: center">
            <div style="flex: 1; color: #00bebd">{{ item.start }} ~ {{ item.end ? item.end : '至今' }}</div>
            <div style="flex: 1; color: #00bebd; font-weight: bold">{{ item.employ }}</div>
            <div style="flex: 1; color: #00bebd; font-weight: bold">{{ item.position }}（{{ item.type }}）</div>
          </div>
          <div style="margin-top: 10px"><strong>{{ item.project }}</strong>：{{ item.content }}</div>
        </div>
        <div></div>
      </div>
      <div style="margin-top: 20px; font-size: 16px" v-if="data.resumeData.proExpList.length">
        <div style="font-weight: bold; padding: 10px; background: linear-gradient(to right, #8edfee, white)">项目经验：</div>
        <div style="margin-top: 10px; margin-bottom: 20px" v-for="item in data.resumeData.proExpList">
          <div style="display: flex; align-items: center">
            <div style="flex: 1; color: #00bebd">{{ item.start }} ~ {{ item.end ? item.end : '至今' }}</div>
            <div style="flex: 1; color: #00bebd; font-weight: bold">{{ item.name }}</div>
          </div>
          <div style="margin-top: 10px">项目介绍：{{ item.content }}</div>
        </div>
        <div></div>
      </div>
    </div>

  </div>
</template>

<script setup>
import {reactive} from "vue";
import request from "@/utils/request.js";
import {ElMessage, ElMessageBox} from "element-plus";
import router from "@/router/index.js";

const data = reactive({
  resumeId: router.currentRoute.value.query.id,
  user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
  resumeData: {
    eduExpList: [],
    workExpList: [],
    proExpList: []
  },
})

const loadResume = () => {
  data.resumeId = router.currentRoute.value.query.id
  if (data.resumeId) {
    request.get('/resume/selectById/' + data.resumeId).then(res => {
      if (res.code === '200') {
        data.resumeData = res.data
      } else {
        ElMessage.error(res.msg)
      }
    })
  }
}
loadResume()
</script>