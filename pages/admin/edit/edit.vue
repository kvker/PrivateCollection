<template>
  <view class="admin-edit">
    <pc-empty-status />

    <!-- 顶部导航 -->
    <view class="nav-header">
      <view class="back-btn" @click="onClickBack">
        <image src="/static/icons/back.png" mode="aspectFit" class="back-icon" />
      </view>
      <text class="nav-title">{{ modeRef === 'view' ? '商品详情' : '编辑商品' }}</text>
    </view>

    <!-- 表单内容 -->
    <scroll-view class="form-content" scroll-y="true">
      <!-- 图片上传 -->
      <view class="form-item">
        <text class="form-label">商品图片</text>
        <view class="image-list">
          <view v-for="(image, index) in formRef.images" :key="index" class="image-item">
            <image :src="image" mode="aspectFill" class="preview-image" @click="onPreviewImage({ index })" />
            <view v-if="isEditableRef" class="delete-btn" @click="onDeleteImage({ index })">×</view>
          </view>
          <view v-if="isEditableRef && formRef.images.length < 9" class="upload-btn" @click="onChooseImage">
            <text class="upload-icon">+</text>
          </view>
        </view>
      </view>

      <!-- 商品名称 -->
      <view class="form-item">
        <text class="form-label">商品名称</text>
        <input v-model="formRef.name" type="text" class="form-input" placeholder="请输入商品名称" :disabled="!isEditableRef" />
      </view>

      <!-- 商品描述 -->
      <view class="form-item">
        <text class="form-label">商品描述</text>
        <textarea v-model="formRef.description" class="form-textarea" placeholder="请输入商品描述"
          :disabled="!isEditableRef" />
      </view>

      <!-- 商品价格 -->
      <view class="form-item">
        <text class="form-label">商品价格</text>
        <input v-model="formRef.price" type="digit" class="form-input" placeholder="请输入商品价格"
          :disabled="!isEditableRef" />
      </view>
    </scroll-view>

    <!-- 底部按钮 -->
    <view v-if="isEditableRef" class="bottom-actions">
      <button class="submit-btn" @click="onSubmitForm">{{ submitTextRef }}</button>
    </view>
  </view>
</template>

<script setup>
import { ref, computed } from 'vue'
import { onLoad } from '@dcloudio/uni-app'
import PcEmptyStatus from '@/components/common/pc-empty-status.vue'

const AV = getApp().globalData.AV

onLoad(async ({ mode, objectId }) => {
  console.log('onLoad', { mode, objectId })
  modeRef.value = mode
  if(objectId) {
    goodsIdRef.value = objectId
    try {
      uni.showLoading({ title: '加载中...' })

      // 查询商品详情
      const query = new AV.Query('Goods')
      const goods = await query.get(objectId)

      // 设置表单数据
      formRef.value = {
        images: goods.get('images') || [],
        name: goods.get('name'),
        description: goods.get('description'),
        price: goods.get('price')
      }

      uni.hideLoading()
    } catch(error) {
      console.error('加载商品详情失败:', error)
      uni.showToast({
        title: '加载失败:' + error.message,
        icon: 'none'
      })
      // 加载失败返回上一页
      setTimeout(() => {
        uni.navigateBack()
      }, 1500)
    }
  }
})

// 页面参数
const modeRef = ref('add') // add, edit, view
const goodsIdRef = ref(null)

// 表单数据
const formRef = ref({
  images: [],
  name: '',
  description: '',
  price: ''
})

// 是否可编辑
const isEditableRef = computed(() => modeRef.value !== 'view')

// 按钮文本
const submitTextRef = computed(() => {
  switch(modeRef.value) {
    case 'add':
      return '提交'
    case 'edit':
      return '更新'
    default:
      return ''
  }
})

// 选择图片
function onChooseImage() {
  if(!isEditableRef.value) return Promise.resolve()

  uni.chooseImage({
    count: 9,
    sizeType: ['compressed'],
    sourceType: ['album', 'camera'],
    success: ({ tempFilePaths }) => {
      // 压缩图片
      Promise.all(tempFilePaths.map(path => compressImage(path)))
        .then(compressedPaths => {
          formRef.value.images.push(...compressedPaths)
        })
    }
  })
  return Promise.resolve()
}

// 压缩图片
function compressImage(path) {
  return new Promise((resolve, reject) => {
    uni.compressImage({
      src: path,
      quality: 80,
      success: ({ tempFilePath }) => {
        // 检查文件大小
        uni.getFileInfo({
          filePath: tempFilePath,
          success: ({ size }) => {
            if(size > 2 * 1024 * 1024) {
              // 如果还是超过2MB，继续压缩
              compressImage(tempFilePath).then(resolve)
            } else {
              resolve(tempFilePath)
            }
          },
          fail: reject
        })
      },
      fail: reject
    })
  })
}

// 删除图片
function onDeleteImage({ index }) {
  if(!isEditableRef.value) return Promise.resolve()
  formRef.value.images.splice(index, 1)
  return Promise.resolve()
}

// 预览图片
function onPreviewImage({ index }) {
  uni.previewImage({
    current: index,
    urls: formRef.value.images
  })
  return Promise.resolve()
}

// 提交表单
async function onSubmitForm() {
  if(!isEditableRef.value) return Promise.resolve()

  // 表单验证
  if(!formRef.value.images.length) {
    uni.showToast({ title: '请上传商品图片', icon: 'none' })
    return Promise.resolve()
  }
  if(!formRef.value.name) {
    uni.showToast({ title: '请输入商品名称', icon: 'none' })
    return Promise.resolve()
  }
  if(!formRef.value.description) {
    uni.showToast({ title: '请输入商品描述', icon: 'none' })
    return Promise.resolve()
  }
  if(!formRef.value.price) {
    uni.showToast({ title: '请输入商品价格', icon: 'none' })
    return Promise.resolve()
  }

  try {
    // 显示加载提示
    uni.showLoading({ title: '提交中...' })

    // 1. 先上传图片到LeanCloud
    const imageFiles = await Promise.all(formRef.value.images.map(async (imagePath) => {
      const file = new AV.File(`goods_${Date.now()}.png`, {
        blob: {
          uri: imagePath,
        }
      })
      return file.save()
    }))

    // 2. 创建商品对象
    const Goods = AV.Object.extend('Goods')
    const goods = new Goods()

    // 3. 设置商品属性
    goods.set('name', formRef.value.name)
    goods.set('description', formRef.value.description)
    goods.set('price', Number(formRef.value.price))
    goods.set('images', imageFiles.map(file => file.url()))

    // 4. 保存商品
    await goods.save()

    // 5. 提示成功
    uni.hideLoading()
    uni.showToast({
      title: '提交成功',
      icon: 'success',
      duration: 1500
    })

    // 6. 返回列表页并刷新
    setTimeout(() => {
      const pages = getCurrentPages()
      const listPage = pages[pages.length - 2]
      if(listPage && listPage.$vm) {
        // 触发列表页的刷新方法
        listPage.$vm.onPullDownRefresh()
      }
      uni.navigateBack()
    }, 1500)

  } catch(error) {
    uni.hideLoading()
    uni.showToast({
      title: '提交失败：' + error.message,
      icon: 'none',
      duration: 2000
    })
    console.error('提交商品失败:', error)
  }

  return Promise.resolve()
}

// 返回上一页
function onClickBack() {
  uni.navigateBack()
  return Promise.resolve()
}
</script>

<style>
.admin-edit {
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
}

.back-btn {
  width: 48rpx;
  height: 48rpx;
  margin-right: 32rpx;
}

.back-icon {
  width: 100%;
  height: 100%;
}

.nav-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.form-content {
  flex: 1;
  padding: 32rpx;
  margin-bottom: 120rpx;
}

.form-item {
  margin-bottom: 32rpx;
}

.form-label {
  display: block;
  font-size: 28rpx;
  color: #333;
  margin-bottom: 16rpx;
}

.image-list {
  display: flex;
  flex-wrap: wrap;
  gap: 16rpx;
}

.image-item {
  position: relative;
  width: 200rpx;
  height: 200rpx;
}

.preview-image {
  width: 100%;
  height: 100%;
  border-radius: 12rpx;
}

.delete-btn {
  position: absolute;
  top: -16rpx;
  right: -16rpx;
  width: 40rpx;
  height: 40rpx;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 20rpx;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32rpx;
}

.upload-btn {
  width: 200rpx;
  height: 200rpx;
  background: #f8f8f8;
  border: 2rpx dashed #ddd;
  border-radius: 12rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.upload-icon {
  font-size: 48rpx;
  color: #999;
}

.form-input {
  width: 100%;
  height: 88rpx;
  background: #fff;
  border-radius: 12rpx;
  padding: 0 24rpx;
  font-size: 28rpx;
}

.form-textarea {
  width: 100%;
  height: 200rpx;
  background: #fff;
  border-radius: 12rpx;
  padding: 24rpx;
  font-size: 28rpx;
}

.bottom-actions {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 32rpx;
  padding: 0 32rpx;
}

.submit-btn {
  width: 100%;
  height: 88rpx;
  background: #07c160;
  border-radius: 44rpx;
  color: #fff;
  font-size: 32rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
