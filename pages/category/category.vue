<template>
  <view class="category-page">
    <!-- 状态栏 -->
    <pc-status></pc-status>

    <!-- 搜索区域 -->
    <view class="search-area">
      <view class="nav-header">
        <view class="back-btn" @click="onClickBack">
          <image src="/static/icons/back.png" class="back-icon" mode="aspectFit" />
        </view>
        <text class="category-title">{{ categoryNameRef }}</text>
      </view>
      <view class="search-box">
        <input type="text" v-model="searchKeywordRef" placeholder="搜索" @confirm="onSearch" />
      </view>
    </view>

    <!-- 商品列表 -->
    <scroll-view class="goods-list" scroll-y="true" refresher-enabled="true" :refresher-triggered="isRefreshingRef"
      @refresherrefresh="onRefresh" @scrolltolower="onLoadMore">
      <view class="goods-grid">
        <view v-for="item in goodsListRef" :key="item.id" class="goods-card" @click="onClickGoods(item)">
          <view class="favorite-btn" @click.stop="onClickFavorite(item)">
            <image :src="item.isFavorite ? '/static/icons/collection-selected.png' : '/static/icons/collection.png'"
              class="favorite-icon" mode="aspectFit"></image>
          </view>
          <image :src="item.image" mode="aspectFill" class="goods-image"></image>
          <view class="goods-info">
            <view class="user-info">
              <image :src="item.userAvatar" mode="aspectFill" class="user-avatar"></image>
              <text class="user-name">{{ item.userName }}</text>
              <text class="user-status">{{ item.userStatus }}</text>
            </view>
            <view class="goods-detail">
              <text class="goods-name">{{ item.name }}</text>
              <text class="goods-price">¥{{ item.price }}</text>
            </view>
          </view>
        </view>
      </view>
      <view v-if="isLoadingRef" class="loading">加载中...</view>
    </scroll-view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import PcStatus from '@/components/common/pc-status.vue'

// 页面参数
const categoryNameRef = ref('')
const searchKeywordRef = ref('')
const isRefreshingRef = ref(false)
const isLoadingRef = ref(false)
const pageRef = ref(1)

// Mock商品数据
const mockGoods = [
  {
    id: 1,
    userName: '一只大白',
    userAvatar: '/static/temp/avatar.png',
    userStatus: '2天前在线',
    name: '19世纪中古壁灯',
    price: 600,
    image: '/static/temp/goods.png',
    isFavorite: false
  },
  {
    id: 2,
    userName: '是个溜溜',
    userAvatar: '/static/temp/avatar.png',
    userStatus: '10天前在线',
    name: '古董大镜子 还能正常用的 适合装修',
    price: 1200,
    image: '/static/temp/goods.png',
    isFavorite: false
  }
]

const goodsListRef = ref([...mockGoods])

// 页面加载
onMounted(() => {
  const pages = getCurrentPages()
  const currentPage = pages[pages.length - 1]
  const { category } = currentPage.options
  categoryNameRef.value = decodeURIComponent(category) || '分类'
})

// 返回上一页
function onClickBack() {
  uni.navigateBack()
  return Promise.resolve()
}

// 搜索
function onSearch() {
  console.log('搜索关键词:', searchKeywordRef.value)
  return Promise.resolve()
}

// 下拉刷新
function onRefresh() {
  isRefreshingRef.value = true
  try {
    // 模拟请求
    setTimeout(() => {
      goodsListRef.value = [...mockGoods]
      pageRef.value = 1
      isRefreshingRef.value = false
    }, 1000)
  } catch(error) {
    isRefreshingRef.value = false
  }
  return Promise.resolve()
}

// 加载更多
function onLoadMore() {
  if(isLoadingRef.value) return Promise.resolve()
  isLoadingRef.value = true
  try {
    // 模拟请求
    setTimeout(() => {
      const newGoods = mockGoods.map(item => ({
        ...item,
        id: item.id + pageRef.value * mockGoods.length
      }))
      goodsListRef.value.push(...newGoods)
      pageRef.value++
      isLoadingRef.value = false
    }, 1000)
  } catch(error) {
    isLoadingRef.value = false
  }
  return Promise.resolve()
}

// 点击商品
function onClickGoods(goods) {
  console.log('点击商品:', goods)
  return Promise.resolve()
}

// 收藏商品
function onClickFavorite({ id }) {
  const item = goodsListRef.value.find(item => item.id === id)
  if(item) {
    item.isFavorite = !item.isFavorite
  }
  return Promise.resolve()
}
</script>

<style>
.category-page {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f5f5f5;
}

.nav-header {
  display: flex;
  align-items: center;
  padding: 0 32rpx 0 0;
}

.back-btn {
  display: flex;
  align-items: center;
}

.back-icon {
  width: 48rpx;
  height: 48rpx;
}

.category-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  margin-left: 16rpx;
}

.search-area {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16rpx;
  margin: 0 32rpx 32rpx 0;
  background: #fff;
  border-radius: 30rpx;
}

.search-box {
  flex: 1;
  padding: 16rpx 32rpx;
}

.search-box input {
  width: 100%;
  height: 60rpx;
  font-size: 28rpx;
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
  position: relative;
}

.favorite-btn {
  position: absolute;
  top: 20rpx;
  right: 20rpx;
  z-index: 10;
}

.favorite-icon {
  width: 40rpx;
  height: 40rpx;
}

.goods-image {
  width: 100%;
  height: 300rpx;
}

.goods-info {
  padding: 20rpx;
}

.user-info {
  display: flex;
  align-items: center;
  margin-bottom: 16rpx;
}

.user-avatar {
  width: 48rpx;
  height: 48rpx;
  border-radius: 24rpx;
  margin-right: 12rpx;
}

.user-name {
  font-size: 24rpx;
  color: #333;
  margin-right: 12rpx;
}

.user-status {
  font-size: 24rpx;
  color: #999;
}

.goods-detail {
  display: flex;
  flex-direction: column;
  gap: 8rpx;
}

.goods-name {
  font-size: 28rpx;
  color: #333;
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