<script setup>
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'
import PcEmptyStatus from '@/components/common/pc-empty-status.vue'
import PcBack from '@/components/common/pc-back.vue'
import PcGoods from '@/components/common/pc-goods.vue'

const AV = getApp().globalData.AV
const PAGE_SIZE = 10

// 页面参数
const categoryNameRef = ref('')
const categoryRef = ref(null)
const searchKeywordRef = ref('')
const isRefreshingRef = ref(false)
const isLoadingRef = ref(false)
const pageRef = ref(1)
const hasMoreRef = ref(true)
const goodsListRef = ref([])

// 查询分类信息
async function queryCategoryDetail(objectId) {
  const query = new AV.Query('Category')
  const category = await query.get(objectId)
  return category.toJSON()
}

// 查询商品列表
async function queryGoodsList(page = 1, keyword = '') {
  const query = new AV.Query('Goods')
  // 按分类查询
  const category = AV.Object.createWithoutData('Category', categoryRef.value.objectId)
  query.equalTo('categoryRef', category)
  // 如果有搜索关键词
  if (keyword) {
    query.contains('name', keyword)
  }
  query.descending('updatedAt')
  query.limit(PAGE_SIZE)
  query.skip((page - 1) * PAGE_SIZE)
  query.include('categoryRef')

  const results = await query.find()
  if (results.length < PAGE_SIZE) {
    hasMoreRef.value = false
  }

  return results.map((item) => item.toJSON())
}

// 页面加载
onLoad(async ({ objectId }) => {
  try {
    // 1. 获取分类信息
    const category = await queryCategoryDetail(objectId)
    categoryRef.value = category
    categoryNameRef.value = category.name

    // 2. 加载商品列表
    const goods = await queryGoodsList()
    goodsListRef.value = goods
  } catch (error) {
    console.error('加载商品列表失败:', error)
    uni.showToast({
      title: '加载失败:' + error.message,
      icon: 'none'
    })
  }
})

// 搜索
async function onSearch(e) {
  const keyword = e.detail.value.trim()
  searchKeywordRef.value = keyword
  // 重置列表并查询
  goodsListRef.value = []
  pageRef.value = 1
  hasMoreRef.value = true
  try {
    const goods = await queryGoodsList(1, keyword)
    goodsListRef.value = goods
  } catch (error) {
    console.error('搜索商品失败:', error)
    uni.showToast({
      title: '搜索失败:' + error.message,
      icon: 'none'
    })
  }
  return Promise.resolve()
}

// 下拉刷新
async function onRefresh() {
  isRefreshingRef.value = true
  try {
    const goods = await queryGoodsList(1, searchKeywordRef.value)
    goodsListRef.value = goods
    pageRef.value = 1
    hasMoreRef.value = goods.length === PAGE_SIZE
  } catch (error) {
    console.error('刷新商品列表失败:', error)
    uni.showToast({
      title: '刷新失败:' + error.message,
      icon: 'none'
    })
  } finally {
    isRefreshingRef.value = false
  }
  return Promise.resolve()
}

// 加载更多
async function onLoadMore() {
  if (!hasMoreRef.value) return Promise.resolve()
  if (isLoadingRef.value) return Promise.resolve()
  isLoadingRef.value = true
  try {
    const nextPage = pageRef.value + 1
    const newGoods = await queryGoodsList(nextPage, searchKeywordRef.value)
    if (newGoods.length > 0) {
      goodsListRef.value.push(...newGoods)
      pageRef.value = nextPage
    }
  } catch (error) {
    console.error('加载更多商品失败:', error)
    uni.showToast({
      title: '加载失败:' + error.message,
      icon: 'none'
    })
  } finally {
    isLoadingRef.value = false
  }
  return Promise.resolve()
}

// 点击商品
function onClickGoods(goods) {
  uni.navigateTo({
    url: `/pages/goods-detail/goods-detail?objectId=${goods.objectId}`
  })
  return Promise.resolve()
}
</script>

<template>
  <view class="category-page">
    <pc-empty-status />

    <!-- 搜索区域 -->
    <view class="search-area">
      <view class="nav-header">
        <pc-back />
        <text class="category-title">{{ categoryNameRef }}</text>
      </view>
      <view class="search-box">
        <input
          type="text"
          v-model="searchKeywordRef"
          placeholder="搜索商品名称"
          @confirm="onSearch" />
      </view>
    </view>

    <!-- 商品列表 -->
    <scroll-view
      class="goods-list"
      scroll-y="true"
      refresher-enabled="true"
      :refresher-triggered="isRefreshingRef"
      @refresherrefresh="onRefresh"
      @scrolltolower="onLoadMore">
      <view class="goods-grid">
        <pc-goods
          v-for="item in goodsListRef"
          :key="item.objectId"
          :goods="item"
          @click="onClickGoods" />
      </view>
      <view
        v-if="goodsListRef.length === 0"
        class="empty-tip">
        暂无商品数据
      </view>
      <view
        v-if="isLoadingRef"
        class="loading">
        加载中...
      </view>
      <view
        v-if="!isLoadingRef && !hasMoreRef"
        class="no-more">
        没有更多数据了
      </view>
    </scroll-view>
  </view>
</template>

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
  height: 112rpx;
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
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 400rpx;
}

.search-area {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16rpx;
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
  /* flex: 1; */
  height: calc(100% - 200rpx);
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
