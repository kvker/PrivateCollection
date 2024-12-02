<script setup>
import { ref } from 'vue'
import { onLoad, onShareAppMessage } from '@dcloudio/uni-app'
import PcEmptyStatus from '@/components/common/pc-empty-status.vue'

const AV = getApp().globalData.AV

// 页面数据
const currentSwiperRef = ref(0)
const goodsRef = ref(null)
const imagesRef = ref([])
const similarGoodsRef = ref([])

// 查询商品详情
async function queryGoodsDetail(objectId) {
  const query = new AV.Query('Goods')
  // 包含分类信息
  query.include('categoryRef')
  const goods = await query.get(objectId)
  return goods.toJSON()
}

// 查询相似商品
async function querySimilarGoods(categoryId, currentGoodsId) {
  const query = new AV.Query('Goods')
  // 同分类
  const category = AV.Object.createWithoutData('Category', categoryId)
  query.equalTo('categoryRef', category)
  // 排除当前商品
  query.notEqualTo('objectId', currentGoodsId)
  // 最多4个
  query.limit(4)
  const results = await query.find()
  return results.map(item => item.toJSON())
}

onLoad(async (options) => {
  console.log('商品详情:', options.objectId)
  try {
    // 1. 加载商品详情
    const goods = await queryGoodsDetail(options.objectId)
    goodsRef.value = goods
    imagesRef.value = goods.images || []

    // 2. 加载相似商品
    if(goods.categoryRef) {
      const similarGoods = await querySimilarGoods(goods.categoryRef.objectId, goods.objectId)
      similarGoodsRef.value = similarGoods
    }
  } catch(error) {
    console.error('加载商品详情失败:', error)
    uni.showToast({
      title: '加载失败:' + error.message,
      icon: 'none'
    })
    setTimeout(() => {
      uni.navigateBack()
    }, 1500)
  }
})

// 事件处理
function onClickBack() {
  uni.navigateBack()
  return Promise.resolve()
}

function onClickShare() {
  // 使用第一张图片作为分享图
  const shareImage = imagesRef.value[0] || ''
  uni.showShareMenu({
    withShareTicket: true,
    menus: ['shareAppMessage']
  })
  return Promise.resolve()
}

// 分享给朋友
onShareAppMessage(() => {
  return {
    title: '有友藏-' + goodsRef.value?.name,
    path: `/pages/goods/goods-detail?objectId=${goodsRef.value?.objectId}`,
    imageUrl: imagesRef.value[0]
  }
})

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
    url: `/pages/goods/goods-detail?objectId=${goods.objectId}`
  })
  return Promise.resolve()
}

function onClickChat() {
  uni.openCustomerServiceChat({
    extInfo: { url: 'https://work.weixin.qq.com/kfid/xxx' }, // 替换为实际的客服URL
    corpId: 'xxxx', // 替换为实际的企业ID
    success(res) {
      console.log('打开客服成功')
    },
    fail(err) {
      console.error('打开客服失败:', err)
      uni.showToast({
        title: '打开客服失败',
        icon: 'none'
      })
    }
  })
  return Promise.resolve()
}
</script>

<template>
  <view class="goods-detail">
    <pc-empty-status />
    <!-- 顶部导航 -->
    <view class="nav-header">
      <view class="back-btn" @click="onClickBack">
        <image src="/static/icons/back.png" mode="aspectFit" class="back-icon" />
      </view>
      <view class="right-actions">
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
        <text class="goods-name">{{ goodsRef?.name }}</text>
        <view class="price-tag">
          <text class="price">¥{{ goodsRef?.price }}</text>
          <text class="tag">{{ goodsRef?.categoryRef?.name }}</text>
        </view>
      </view>

      <!-- 商品描述 -->
      <view class="goods-desc">
        <view class="section-title">商品描述</view>
        <text class="desc-text">{{ goodsRef?.description }}</text>
      </view>

      <!-- 相似物品 -->
      <view v-if="similarGoodsRef.length > 0" class="similar-goods">
        <view class="section-title">相似的物品</view>
        <scroll-view class="similar-list" scroll-x="true">
          <view v-for="item in similarGoodsRef" :key="item.objectId" class="similar-item"
            @click="onClickSimilarGoods(item)">
            <image :src="item.images[0]" mode="aspectFill" class="similar-image" />
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
      <button class="chat-btn full-width" @click="onClickChat">聊一聊</button>
    </view>
  </view>
</template>

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
  margin-bottom: 240rpx;
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
  margin-bottom: 16rpx;
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
  margin-bottom: 16rpx;
}

.desc-text {
  font-size: 28rpx;
  color: #666;
  line-height: 1.6;
}

.similar-goods {
  background: #fff;
  padding: 32rpx;
  margin-bottom: 320rpx;
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
  bottom: 64rpx;
  height: 120rpx;
  display: flex;
  align-items: center;
  justify-content: center;
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

.full-width {
  width: 100%;
}

.chat-btn {
  background: #07c160;
  color: #fff;
}
</style>