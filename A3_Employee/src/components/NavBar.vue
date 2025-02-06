<template>
  <el-menu :default-active="activeIndex" class="navbar" mode="horizontal" @select="handleSelect">
    <el-menu-item index="1" @click="navigateTo('/')">Companies</el-menu-item>
    <el-menu-item index="2" @click="navigateTo('/employees')">Employees</el-menu-item>
    <el-menu-item index="3" @click="navigateTo('/time-logs')">Time Logs</el-menu-item>
  </el-menu>
  <div class="h-6" />
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const activeIndex = ref('1')
const router = useRouter()

const handleSelect = (key: string, keyPath: string[]) => {
  console.log(key, keyPath)
}

const navigateTo = (path: string) => {
  router.push(path)
}
</script>

<style scoped>
.navbar {
  background-color: rgb(244, 207, 212);
  border-bottom: 2px solid #ff1493;
  padding: 10px 20px;
  width: 1750px;
  margin-top: 20px;
  border-radius: 30px;
  height: 70px;
}

.el-menu-item {
  color: #ff1493;
  font-size: 18px;
}

.el-menu-item:hover {
  background-color: #ff1493;
  color: white;
}

.el-menu-item.is-active {
  color: #ff1493;
}
</style>
