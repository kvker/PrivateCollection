<script setup>
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'
import PcEmptyStatus from '@/components/common/pc-empty-status.vue'

const AV = getApp().globalData.AV
const PAGE_SIZE = 10

// 页面数据
const goodsListRef = ref([])
const pageRef = ref(1)
const isLoadingRef = ref(false)
const isRefreshingRef = ref(false)
const hasMoreRef = ref(true)

// 查询商品列表
async function queryGoodsList(page = 1) {
  const query = new AV.Query('Goods')
  query.descending('createdAt')
  query.limit(PAGE_SIZE)
  query.skip((page - 1) * PAGE_SIZE)

  const results = await query.find()

  if(results.length < PAGE_SIZE) {
    hasMoreRef.value = false
  }

  return results.map(item => item.toJSON())
}

onLoad(async () => {
  try {
    const goods = await queryGoodsList()
    goodsListRef.value = goods
  } catch(error) {
    console.error('加载商品列表失败:', error)
    uni.showToast({
      title: '加载失败:' + error.message,
      icon: 'none'
    })
  }
})

// 下拉刷新
async function onRefresh() {
  if(isRefreshingRef.value) return
  isRefreshingRef.value = true
  try {
    const goods = await queryGoodsList()
    goodsListRef.value = goods
    pageRef.value = 1
    hasMoreRef.value = true
  } catch(error) {
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

// 上拉加载
async function onLoadMore() {
  if(!hasMoreRef.value) return Promise.resolve()
  if(isLoadingRef.value) return Promise.resolve()

  isLoadingRef.value = true
  try {
    const nextPage = pageRef.value + 1
    const newGoods = await queryGoodsList(nextPage)
    if(newGoods.length > 0) {
      goodsListRef.value.push(...newGoods)
      pageRef.value = nextPage
    }
  } catch(error) {
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

// 删除商品
async function onDeleteGoods({ objectId, name }) {
  try {
    const { confirm } = await uni.showModal({
      title: '提示',
      content: `确定要删除 ${name} 这个商品吗？`,
      confirmText: '删除',
      confirmColor: '#ff4d4f'
    })
    if(!confirm) return

    uni.showLoading({ title: '删除中...' })

    // 从 LeanCloud 删除
    const goods = AV.Object.createWithoutData('Goods', objectId)
    await goods.destroy()

    // 从列表中移除
    goodsListRef.value = goodsListRef.value.filter(item => item.objectId !== objectId)

    uni.hideLoading()
    uni.showToast({
      title: '删除成功',
      icon: 'success'
    })
  } catch(error) {
    if(error.errMsg?.includes('cancel')) return

    console.error('删除商品失败:', error)
    uni.showToast({
      title: '删除失败:' + error.message,
      icon: 'none'
    })
  }
  return Promise.resolve()
}

// 点击商品
function onClickGoods({ objectId }) {
  uni.navigateTo({
    url: `/pages/admin/edit/edit?objectId=${objectId}&mode=view`
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
    <scroll-view
      class="goods-list"
      scroll-y="true"
      refresher-enabled="true"
      :refresher-triggered="isRefreshingRef"
      @refresherrefresh="onRefresh"
      @scrolltolower="onLoadMore"
    >
      <view class="goods-grid">
        <view v-for="item in goodsListRef"
          :key="item.objectId"
          class="goods-card"
          @click="onClickGoods(item)"
          @longpress="onDeleteGoods(item)"
        >
          <image :src="item.images[0]" mode="aspectFill" class="goods-image" />
          <view class="goods-info">
            <text class="goods-name">{{ item.name }}</text>
            <text class="goods-price">¥{{ item.price }}</text>
          </view>
        </view>
      </view>

      <!-- 加载状态 -->
      <view v-if="goodsListRef.length === 0" class="empty-tip">
        暂无商品数据
      </view>
      <view v-if="isLoadingRef" class="loading">
        加载中...
      </view>
      <view v-if="!isLoadingRef && !hasMoreRef" class="no-more">
        没有更多数据了
      </view>
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

.no-more {
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
</style>
