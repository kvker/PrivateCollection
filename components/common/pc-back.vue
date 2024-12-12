<script setup>
import { ref } from 'vue'

const openType = ref('navigateBack')
const currentPage = getCurrentPages()
if (currentPage.length > 1) {
  openType.value = 'navigateBack'
} else {
  openType.value = 'reLaunch'
}
</script>

<template>
  <navigator
    class="back-btn"
    :open-type="openType"
    url="/pages/index/index">
    <image
      src="/static/icons/back.png"
      mode="aspectFit"
      class="back-icon" />
  </navigator>
</template>

<style>
.back-btn {
  width: 48rpx;
  height: 48rpx;
}

.back-icon {
  width: 100%;
  height: 100%;
}
</style>
