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

onMounted(() => {

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
