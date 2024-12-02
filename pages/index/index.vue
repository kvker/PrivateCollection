<script setup>
import { ref } from 'vue'
import PcStatus from '@/components/common/pc-status.vue'

// 分类数据
const categoriesRef = ref([
  { id: 1, name: '家居', icon: '/static/icons/home.png' },
  { id: 2, name: '手作', icon: '/static/icons/handmade.png' },
  { id: 3, name: 'ACG', icon: '/static/icons/acg.png' },
  { id: 4, name: '数码', icon: '/static/icons/digital.png' },
  { id: 5, name: '潮玩', icon: '/static/icons/toy.png' },
  { id: 6, name: '古玩', icon: '/static/icons/antique.png' },
  { id: 7, name: '收藏', icon: '/static/icons/collection.png' },
  { id: 8, name: '其他', icon: '/static/icons/other.png' }
])

// 最近好物数据
const mockItems = [
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
  },
  {
    id: 3,
    userName: '淘淘淘乐',
    userAvatar: '/static/temp/avatar.png',
    userStatus: '3小时前在线',
    name: '老式座钟 八音盒 可正常使用',
    price: 3600,
    image: '/static/temp/goods.png',
    isFavorite: false
  },
  {
    id: 4,
    userName: '李沐沐',
    userAvatar: '/static/temp/avatar.png',
    userStatus: '刚刚在线',
    name: '民国老茶几 纯实木',
    price: 2800,
    image: '/static/temp/goods.png',
    isFavorite: false
  },
  {
    id: 3,
    userName: '淘淘淘乐',
    userAvatar: '/static/temp/avatar.png',
    userStatus: '3小时前在线',
    name: '老式座钟 八音盒 可正常使用',
    price: 3600,
    image: '/static/temp/goods.png',
    isFavorite: false
  },
  {
    id: 4,
    userName: '李沐沐',
    userAvatar: '/static/temp/avatar.png',
    userStatus: '刚刚在线',
    name: '民国老茶几 纯实木',
    price: 2800,
    image: '/static/temp/goods.png',
    isFavorite: false
  },
  {
    id: 3,
    userName: '淘淘淘乐',
    userAvatar: '/static/temp/avatar.png',
    userStatus: '3小时前在线',
    name: '老式座钟 八音盒 可正常使用',
    price: 3600,
    image: '/static/temp/goods.png',
    isFavorite: false
  },
  {
    id: 4,
    userName: '李沐沐',
    userAvatar: '/static/temp/avatar.png',
    userStatus: '刚刚在线',
    name: '民国老茶几 纯实木',
    price: 2800,
    image: '/static/temp/goods.png',
    isFavorite: false
  }
]

const recentItemsRef = ref([...mockItems])
const isRefreshingRef = ref(false)
const isLoadingRef = ref(false)
const pageRef = ref(1)

// 下拉刷新
const onRefresh = async () => {
  isRefreshingRef.value = true
  try {
    // 模拟请求
    await new Promise(resolve => setTimeout(resolve, 1000))
    recentItemsRef.value = [...mockItems]
    pageRef.value = 1
  } finally {
    isRefreshingRef.value = false
  }
  return Promise.resolve()
}

// 上拉加载更多
const onLoadMore = async () => {
  if(isLoadingRef.value) return Promise.resolve()
  isLoadingRef.value = true
  try {
    // 模拟请求
    await new Promise(resolve => setTimeout(resolve, 1000))
    const newItems = mockItems.map(item => ({
      ...item,
      id: item.id + pageRef.value * 4
    }))
    recentItemsRef.value.push(...newItems)
    pageRef.value++
  } finally {
    isLoadingRef.value = false
  }
  return Promise.resolve()
}

// 点击分类
function onClickCategory(category) {
  uni.navigateTo({
    url: `/pages/category/category?category=${category.name}`
  })
  return Promise.resolve()
}

// 点击商品
const onClickItem = (item) => {
  console.log('点击商品:', item)
  // TODO: 实现商品详情跳转
  return Promise.resolve()
}

// 添加收藏点击事件
function onClickFavorite({ id }) {
  const item = recentItemsRef.value.find(item => item.id === id)
  if(item) {
    item.isFavorite = !item.isFavorite
  }
  return Promise.resolve()
}
</script>

<template>
  <view class="home-page">
    <!-- 状态栏 -->
    <pc-status>
      <text class="status-title">私藏</text>
    </pc-status>

    <!-- 内容区域 -->
    <view class="content-area">
      <!-- 搜索框 -->
      <view class="search-box">
        <input type="text" placeholder="搜索" />
      </view>

      <!-- 分类导航 - 横向滚动 -->
      <scroll-view class="category-scroll" scroll-x="true" show-scrollbar="false">
        <view class="category-nav">
          <view v-for="category in categoriesRef" :key="category.id" class="category-item"
            @click="onClickCategory(category)">
            <image :src="category.icon" mode="aspectFit" class="category-icon"></image>
            <text class="category-name">{{ category.name }}</text>
          </view>
        </view>
      </scroll-view>

      <!-- 最近好物 - 上下滚动 -->
      <scroll-view class="recent-items" scroll-y="true" refresher-enabled="true" :refresher-triggered="isRefreshingRef"
        @refresherrefresh="onRefresh" @scrolltolower="onLoadMore">
        <view class="section-title">最近好物</view>
        <view class="items-grid">
          <view v-for="item in recentItemsRef" :key="item.id" class="item-card" @click="onClickItem(item)">
            <view class="favorite-btn" @click.stop="onClickFavorite(item)">
              <image :src="item.isFavorite ? '/static/icons/collection-selected.png' : '/static/icons/collection.png'"
                class="favorite-icon" mode="aspectFit" />
            </view>
            <image :src="item.image" mode="aspectFill" class="item-image"></image>
            <view class="item-info">
              <view class="item-user">
                <image :src="item.userAvatar" mode="aspectFill" class="user-avatar"></image>
                <text class="user-name">{{ item.userName }}</text>
                <text class="user-status">{{ item.userStatus }}</text>
              </view>
              <view class="item-detail">
                <text class="item-name">{{ item.name }}</text>
                <text class="item-price">¥{{ item.price }}</text>
              </view>
            </view>
          </view>
        </view>
        <view v-if="isLoadingRef" class="loading">加载中...</view>
      </scroll-view>
    </view>
  </view>
</template>

<style>
.home-page {
  width: 100%;
  height: 100vh;
}

.status-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.content-area {
  height: calc(100% - 88rpx);
  padding: 20rpx;
}

.search-box {
  padding: 20rpx;
  background: #f5f5f5;
  border-radius: 40rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 64rpx;
  margin-bottom: 32rpx;
}

.search-box input {
  width: 100%;
  height: 32rpx;
  font-size: 28rpx;
}

.category-scroll {
  width: 100%;
  white-space: nowrap;
  margin-bottom: 32rpx;
}

.category-nav {
  display: inline-flex;
}

.category-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 120rpx;
  height: 120rpx;
  aspect-ratio: 1;
  flex-shrink: 0;
}

.category-icon {
  width: 80rpx;
  height: 80rpx;
  margin-bottom: 10rpx;
}

.category-name {
  font-size: 24rpx;
  color: #333;
}

.recent-items {
  height: calc(100% - 248rpx);
}

.section-title {
  font-size: 32rpx;
  font-weight: bold;
  margin-bottom: 20rpx;
}

.items-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20rpx;
}

.item-card {
  background: #fff;
  border-radius: 20rpx;
  overflow: hidden;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.1);
  position: relative;
}

.item-user {
  display: flex;
  align-items: center;
  padding: 20rpx 0;
}

.user-avatar {
  width: 60rpx;
  height: 60rpx;
  border-radius: 30rpx;
  margin-right: 10rpx;
}

.user-name {
  font-size: 26rpx;
  color: #333;
  margin-right: 10rpx;
}

.user-status {
  font-size: 24rpx;
  color: #999;
}

.item-image {
  width: 100%;
  height: 300rpx;
}

.item-info {
  padding: 20rpx;
}

.item-name {
  font-size: 28rpx;
  color: #333;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

.item-price {
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

.item-detail {
  display: flex;
  flex-direction: column;
  gap: 8rpx;
}

.item-name {
  font-size: 28rpx;
  color: #333;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

.item-price {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
}
</style>
