<template>
  <view class="pc-status" :style="{ backgroundColor: bgColor }">
    <view class="pc-status-content">
      <slot></slot>
    </view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  bgColor: {
    type: String,
    default: 'transparent'
  }
})

const statusBarHeight = ref('44px')

onMounted(() => {
  // 获取系统状态栏高度
  uni.getSystemInfoSync().statusBarHeight
})
</script>

<style>
.pc-status-content {
  width: 100%;
  height: 88rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
