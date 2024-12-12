<script setup>
// 商品卡片组件
const props = defineProps({
  // 商品数据
  goods: {
    type: Object,
    required: true
  },
  // 是否显示管理操作(长按删除)
  isAdmin: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['click', 'delete'])

function onClickGoods() {
  emit('click', props.goods)
  return Promise.resolve()
}

function onLongPressGoods() {
  if (!props.isAdmin) return Promise.resolve()
  emit('delete', props.goods)
  return Promise.resolve()
}
</script>

<template>
  <view
    class="goods-card"
    @click="onClickGoods"
    @longpress="onLongPressGoods">
    <image
      :src="goods.images[0]"
      mode="aspectFill"
      class="goods-image" />
    <view class="goods-info">
      <text class="goods-name">{{ goods.name }}</text>
      <text class="goods-price">¥{{ goods.price }}</text>
    </view>
  </view>
</template>

<style>
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
</style>