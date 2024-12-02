<script setup>
import { ref } from 'vue'
import { onLoad, onReachBottom, onPullDownRefresh } from '@dcloudio/uni-app'
import PcEmptyStatus from '@/components/common/pc-empty-status.vue'

// 页面数据
const goodsListRef = ref([])
const pageRef = ref(1)
const isLoadingRef = ref(false)
const isRefreshingRef = ref(false)

// Mock数据
const mockGoods = [
  {
    id: 1,
    name: '19世纪中古壁灯',
    price: 600,
    image: '/static/temp/goods.png'
  },
  {
    id: 2,
    name: '古董大镜子 还能正常用的 适合装修',
    price: 1200,
    image: '/static/temp/goods.png'
  }
]

onLoad(() => {
  goodsListRef.value = [...mockGoods]
})

// 下拉刷新
onPullDownRefresh(() => {
  isRefreshingRef.value = true
  try {
    goodsListRef.value = [...mockGoods]
    pageRef.value = 1
  } finally {
    uni.stopPullDownRefresh()
    isRefreshingRef.value = false
  }
  return Promise.resolve()
})

// 上拉加载
onReachBottom(() => {
  if(isLoadingRef.value) return Promise.resolve()
  isLoadingRef.value = true
  try {
    const newGoods = mockGoods.map(item => ({
      ...item,
      id: item.id + pageRef.value * mockGoods.length
    }))
    goodsListRef.value.push(...newGoods)
    pageRef.value++
  } finally {
    isLoadingRef.value = false
  }
  return Promise.resolve()
})

// 点击商品
function onClickGoods({ id }) {
  uni.navigateTo({
    url: `/pages/admin/edit/edit?id=${id}&mode=view`
  })
  return Promise.resolve()
}

// 点击新增
function onClickAdd() {
  uni.navigateTo({
    url: '/pages/admin/edit/edit?mode=add'
  })
  return Promise.resolve()
}
</script>

<template>
  <view class="admin-list">
    <pc-empty-status />

    <!-- 顶部导航 -->
    <view class="nav-header">
      <text class="nav-title">商品管理</text>
      <view class="add-btn" @click="onClickAdd">
        <text class="add-text">新增</text>
      </view>
    </view>

    <!-- 商品列表 -->
    <scroll-view class="goods-list" scroll-y="true" refresher-enabled="true" :refresher-triggered="isRefreshingRef"
      @refresherrefresh="onPullDownRefresh" @scrolltolower="onReachBottom">
      <view class="goods-grid">
        <view v-for="item in goodsListRef" :key="item.id" class="goods-card" @click="onClickGoods(item)">
          <image :src="item.image" mode="aspectFill" class="goods-image" />
          <view class="goods-info">
            <text class="goods-name">{{ item.name }}</text>
            <text class="goods-price">¥{{ item.price }}</text>
          </view>
        </view>
      </view>
      <view v-if="isLoadingRef" class="loading">加载中...</view>
    </scroll-view>
  </view>
</template>

<style>
.admin-list {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f5f5f5;
}

.nav-header {
  margin-top: 88rpx;
  padding: 32rpx;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.nav-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.add-btn {
  padding: 12rpx 32rpx;
  background: #07c160;
  border-radius: 32rpx;
}

.add-text {
  font-size: 28rpx;
  color: #fff;
}

.goods-list {
  flex: 1;
  padding: 20rpx;
}

.goods-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20rpx;
}

.goods-card {
  background: #fff;
  border-radius: 20rpx;
  overflow: hidden;
}

.goods-image {
  width: 100%;
  height: 300rpx;
}

.goods-info {
  padding: 20rpx;
}

.goods-name {
  font-size: 28rpx;
  color: #333;
  margin-bottom: 8rpx;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

.goods-price {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
}

.loading {
  text-align: center;
  padding: 20rpx;
  color: #999;
  font-size: 24rpx;
}
</style>
