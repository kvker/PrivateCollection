<script setup>
import { ref } from 'vue'
import { onLoad, onShareAppMessage } from '@dcloudio/uni-app'
import PcEmptyStatus from '@/components/common/pc-empty-status.vue'

const AV = getApp().globalData.AV
const PAGE_SIZE = 10

// 分类数据
const categoriesRef = ref([])

// 商品数据
const recentItemsRef = ref([])
const isRefreshingRef = ref(false)
const isLoadingRef = ref(false)
const pageRef = ref(1)
const hasMoreRef = ref(true)
const searchKeywordRef = ref('')

// 查询分类列表
async function queryCategoryList() {
  const query = new AV.Query('Category')
  query.ascending('sort')
  const results = await query.find()
  return results.map(item => item.toJSON())
}

// 查询商品列表
async function queryGoodsList(page = 1, keyword = '') {
  const query = new AV.Query('Goods')
  query.descending('updatedAt')
  query.limit(PAGE_SIZE)
  query.skip((page - 1) * PAGE_SIZE)
  // 如果有搜索关键词
  if(keyword) {
    query.contains('name', keyword)
  }
  // 包含分类信息
  query.include('categoryRef')

  const results = await query.find()
  if(results.length < PAGE_SIZE) {
    hasMoreRef.value = false
  }

  return results.map(item => item.toJSON())
}

// 下拉刷新
const onRefresh = async () => {
  isRefreshingRef.value = true
  try {
    const goods = await queryGoodsList(1, searchKeywordRef.value)
    recentItemsRef.value = goods
    pageRef.value = 1
    hasMoreRef.value = goods.length === PAGE_SIZE
  } finally {
    isRefreshingRef.value = false
  }
  return Promise.resolve()
}

// 上拉加载更多
const onLoadMore = async () => {
  if(!hasMoreRef.value) return Promise.resolve()
  if(isLoadingRef.value) return Promise.resolve()
  isLoadingRef.value = true
  try {
    const nextPage = pageRef.value + 1
    const newGoods = await queryGoodsList(nextPage, searchKeywordRef.value)
    if(newGoods.length > 0) {
      recentItemsRef.value.push(...newGoods)
      pageRef.value = nextPage
    }
  } finally {
    isLoadingRef.value = false
  }
  return Promise.resolve()
}

// 搜索功能
async function onSearch(e) {
  const keyword = e.detail.value.trim()
  searchKeywordRef.value = keyword
  // 重置列表并查询
  recentItemsRef.value = []
  pageRef.value = 1
  hasMoreRef.value = true
  try {
    const goods = await queryGoodsList(1, keyword)
    recentItemsRef.value = goods
  } catch(error) {
    console.error('搜索商品失败:', error)
    uni.showToast({
      title: '搜索失败:' + error.message,
      icon: 'none'
    })
  }
  return Promise.resolve()
}

// 页面加载
onLoad(async () => {
  try {
    // 1. 加载分类
    const categories = await queryCategoryList()
    categoriesRef.value = categories

    // 2. 加载商品列表
    const goods = await queryGoodsList()
    recentItemsRef.value = goods
  } catch(error) {
    console.error('加载数据失败:', error)
    uni.showToast({
      title: '加载失败:' + error.message,
      icon: 'none'
    })
  }
})

// 点击商品
const onClickItem = (item) => {
  console.log('点击商品:', item)
  // TODO: 实现商品详情跳转
  uni.navigateTo({
    url: `/pages/goods-detail/goods-detail?objectId=${item.objectId}`
  })
  return Promise.resolve()
}

// TODO: 添加管理员权限判断条件
function onClickAdmin() {
  uni.navigateTo({
    url: '/pages/admin/list/list'
  })
  return Promise.resolve()
}

// 分享给朋友
onShareAppMessage(() => {
  // 使用列表第一项的第一张图片
  const firstImage = recentItemsRef.value[0]?.images[0] || ''
  return {
    title: '有友藏-这里有神奇的好东西',
    path: '/pages/index/index',
    imageUrl: firstImage
  }
})
</script>

<template>
  <view class="home-page">
    <pc-empty-status />

    <!-- 内容区域 -->
    <view class="content-area">
      <!-- 搜索框 -->
      <view class="search-box">
        <input type="text" placeholder="搜索商品名称" :value="searchKeywordRef" @confirm="onSearch" />
      </view>

      <!-- 分类导航 -->
      <scroll-view class="category-scroll" scroll-x="true" show-scrollbar="false">
        <view class="category-nav">
          <navigator v-for="category in categoriesRef"
            :key="category.objectId"
            class="category-item"
            :url="`/pages/category/category?objectId=${category.objectId}`"
          >
            <image :src="category.icon || '/static/icons/toy.png'" mode="aspectFit" class="category-icon"></image>
            <text class="category-name">{{ category.name }}</text>
          </navigator>
        </view>
      </scroll-view>

      <!-- 商品列表 -->
      <scroll-view class="recent-items" scroll-y="true" refresher-enabled="true" :refresher-triggered="isRefreshingRef"
        @refresherrefresh="onRefresh" @scrolltolower="onLoadMore">
        <view class="section-title">最近好物</view>
        <view class="items-grid">
          <view v-for="item in recentItemsRef" :key="item.objectId" class="item-card" @click="onClickItem(item)">
            <image :src="item.images[0]" mode="aspectFill" class="item-image"></image>
            <view class="item-info">
              <text class="item-name">{{ item.name }}</text>
              <text class="item-price">¥{{ item.price }}</text>
            </view>
          </view>
        </view>
        <view v-if="recentItemsRef.length === 0" class="empty-tip">
          暂无商品数据
        </view>
        <view v-if="isLoadingRef" class="loading">加载中...</view>
        <view v-if="!isLoadingRef && !hasMoreRef" class="no-more">
          没有更多数据了
        </view>
      </scroll-view>
    </view>

    <!-- 管理入口 -->
    <view v-if="true" class="admin-fab" @click="onClickAdmin">
      <image src="/static/icons/add.png" mode="aspectFit" class="admin-icon" />
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
  display: flex;
  justify-content: space-between;
  padding: 20rpx 0;
  margin-bottom: 30rpx;
}

.category-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 140rpx;
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
  height: calc(100% - 284rpx);
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

.admin-fab {
  position: fixed;
  right: 32rpx;
  bottom: 128rpx;
  width: 88rpx;
  height: 88rpx;
  background: #07c160;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.15);
}

.admin-icon {
  width: 44rpx;
  height: 44rpx;
}

.empty-tip {
  text-align: center;
  padding: 40rpx;
  color: #999;
  font-size: 28rpx;
}

.no-more {
  text-align: center;
  padding: 20rpx;
  color: #999;
  font-size: 24rpx;
}
</style>