<template>
  <view class="goods-detail">
    <pc-empty-status />
    <!-- 顶部导航 -->
    <view class="nav-header">
      <view class="back-btn" @click="onClickBack">
        <image src="/static/icons/back.png" mode="aspectFit" class="back-icon" />
      </view>
      <view class="right-actions">
        <view class="favorite-btn" @click="onClickFavorite">
          <image :src="isFavoriteRef ? '/static/icons/collection-selected.png' : '/static/icons/collection.png'"
            mode="aspectFit" class="action-icon" />
          <text class="favorite-count">{{ favoriteCountRef }}</text>
        </view>
        <view class="share-btn" @click="onClickShare">
          <image src="/static/icons/share.png" mode="aspectFit" class="action-icon" />
        </view>
      </view>
    </view>

    <!-- 内容区域 -->
    <scroll-view class="content-area" scroll-y="true">
      <!-- 轮播图 -->
      <swiper class="swiper" circular :current="currentSwiperRef" @change="onSwiperChange">
        <swiper-item v-for="(image, index) in imagesRef" :key="index" @click="onPreviewImage(index)">
          <image :src="image" mode="aspectFill" class="swiper-image" />
        </swiper-item>
      </swiper>
      <view class="swiper-indicator">
        <text class="indicator-text">{{ currentSwiperRef + 1 }}/{{ imagesRef.length }}</text>
      </view>

      <!-- 商品信息 -->
      <view class="goods-info">
        <text class="goods-name">{{ goodsRef.name }}</text>
        <view class="price-tag">
          <text class="price">¥{{ goodsRef.price }}</text>
          <text class="tag">{{ goodsRef.tag }}</text>
        </view>
      </view>

      <!-- 商品描述 -->
      <view class="goods-desc">
        <text class="desc-text">{{ goodsRef.description }}</text>
      </view>

      <!-- 交易信息 -->
      <view class="trade-info">
        <view class="trade-item">
          <text class="trade-label">交易方式</text>
          <text class="trade-value">{{ goodsRef.tradeType }}</text>
        </view>
        <view class="trade-item">
          <text class="trade-label">所在位置</text>
          <text class="trade-value">{{ goodsRef.location }}</text>
        </view>
      </view>

      <!-- 相似物品 -->
      <view class="similar-goods">
        <text class="section-title">相似的物品</text>
        <scroll-view class="similar-list" scroll-x="true">
          <view v-for="item in similarGoodsRef" :key="item.id" class="similar-item" @click="onClickSimilarGoods(item)">
            <image :src="item.image" mode="aspectFill" class="similar-image" />
            <view class="similar-info">
              <text class="similar-name">{{ item.name }}</text>
              <text class="similar-price">¥{{ item.price }}</text>
            </view>
          </view>
        </scroll-view>
      </view>
    </scroll-view>

    <!-- 底部操作栏 -->
    <view class="bottom-actions">
      <button class="chat-btn" @click="onClickChat">聊一聊</button>
      <button class="offer-btn" @click="onClickOffer">出价</button>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import PcEmptyStatus from '@/components/common/pc-empty-status.vue'

// 页面数据
const currentSwiperRef = ref(0)
const isFavoriteRef = ref(false)
const favoriteCountRef = ref(32)

// Mock数据
const imagesRef = ref([
  '/static/temp/goods.png',
  '/static/temp/goods.png',
  '/static/temp/goods.png'
])

const goodsRef = ref({
  name: '初音未来手办 日本制万代正版 2019年 个人收藏绝版',
  price: 50,
  tag: '全新',
  description: '这款赛博朋克小姐，以现实未来情怀重塑，特别定制。日常小姐是绝对正版商品，都能影响不大不过品味，为生活添一件独特收藏。',
  tradeType: '面交',
  location: '杭州西湖区龙坞地铁站'
})

const similarGoodsRef = ref([
  {
    id: 1,
    name: '19世纪中古壁灯',
    price: 600,
    image: '/static/temp/goods.png'
  },
  {
    id: 2,
    name: '古董相机',
    price: 600,
    image: '/static/temp/goods.png'
  }
])

// 事件处理
function onClickBack() {
  uni.navigateBack()
  return Promise.resolve()
}

function onClickFavorite() {
  isFavoriteRef.value = !isFavoriteRef.value
  favoriteCountRef.value += isFavoriteRef.value ? 1 : -1
  return Promise.resolve()
}

function onClickShare() {
  uni.showShareMenu({
    withShareTicket: true
  })
  return Promise.resolve()
}

function onSwiperChange(e) {
  currentSwiperRef.value = e.detail.current
  return Promise.resolve()
}

function onPreviewImage(index) {
  uni.previewImage({
    current: index,
    urls: imagesRef.value
  })
  return Promise.resolve()
}

function onClickSimilarGoods(goods) {
  uni.navigateTo({
    url: `/pages/goods/goods-detail?id=${goods.id}`
  })
  return Promise.resolve()
}

function onClickChat() {
  console.log('点击聊一聊')
  return Promise.resolve()
}

function onClickOffer() {
  console.log('点击出价')
  return Promise.resolve()
}
</script>

<style>
.goods-detail {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f5f5f5;
}

.nav-header {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 32rpx;
}

.back-btn {
  width: 48rpx;
  height: 48rpx;
}

.back-icon {
  width: 100%;
  height: 100%;
}

.right-actions {
  display: flex;
  align-items: center;
  gap: 32rpx;
}

.favorite-btn,
.share-btn {
  display: flex;
  align-items: center;
}

.action-icon {
  width: 48rpx;
  height: 48rpx;
}

.favorite-count {
  font-size: 24rpx;
  color: #fff;
  margin-left: 8rpx;
}

.content-area {
  position: relative;
  flex: 1;
  margin-bottom: 120rpx;
}

.swiper {
  width: 100%;
  height: 750rpx;
}

.swiper-image {
  width: 100%;
  height: 100%;
}

.swiper-indicator {
  position: absolute;
  right: 32rpx;
  top: 680rpx;
  background: rgba(0, 0, 0, 0.5);
  padding: 8rpx 16rpx;
  border-radius: 32rpx;
}

.indicator-text {
  color: #fff;
  font-size: 24rpx;
}

.goods-info {
  background: #fff;
  padding: 32rpx;
  margin-bottom: 20rpx;
}

.goods-name {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
  margin-bottom: 16rpx;
}

.price-tag {
  display: flex;
  align-items: center;
  gap: 16rpx;
}

.price {
  font-size: 40rpx;
  color: #333;
  font-weight: bold;
}

.tag {
  font-size: 24rpx;
  color: #666;
  padding: 4rpx 16rpx;
  background: #f5f5f5;
  border-radius: 8rpx;
}

.goods-desc {
  background: #fff;
  padding: 32rpx;
  margin-bottom: 20rpx;
}

.desc-text {
  font-size: 28rpx;
  color: #666;
  line-height: 1.6;
}

.trade-info {
  background: #fff;
  padding: 32rpx;
  margin-bottom: 20rpx;
}

.trade-item {
  display: flex;
  align-items: center;
  margin-bottom: 16rpx;
}

.trade-item:last-child {
  margin-bottom: 0;
}

.trade-label {
  font-size: 28rpx;
  color: #999;
  width: 140rpx;
}

.trade-value {
  font-size: 28rpx;
  color: #333;
}

.similar-goods {
  background: #fff;
  padding: 32rpx;
}

.section-title {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
  margin-bottom: 24rpx;
}

.similar-list {
  white-space: nowrap;
}

.similar-item {
  display: inline-block;
  margin-right: 24rpx;
  width: 240rpx;
}

.similar-item:last-child {
  margin-right: 0;
}

.similar-image {
  width: 240rpx;
  height: 240rpx;
  border-radius: 12rpx;
  margin-bottom: 12rpx;
}

.similar-info {
  padding: 0 8rpx;
}

.similar-name {
  font-size: 28rpx;
  color: #333;
  margin-bottom: 8rpx;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  display: block;
}

.similar-price {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
}

.bottom-actions {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 128rpx;
  height: 120rpx;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 32rpx;
}

.chat-btn,
.offer-btn {
  width: 320rpx;
  height: 80rpx;
  border-radius: 40rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32rpx;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
}

.chat-btn {
  background: #f5f5f5;
  color: #333;
}

.offer-btn {
  background: #07c160;
  color: #fff;
}
</style>