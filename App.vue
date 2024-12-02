<script>
export default {
  onLaunch() {
    console.log('App Launch')
  },
  onShow() {
    console.log('App Show')
  },
  onHide() {
    console.log('App Hide')
  }
}
</script>

<style>
/*每个页面公共css */
page {
  background-color: #F1F2F4;
}

/* CSS Reset */
view,
text,
scroll-view,
swiper,
button,
form,
input,
textarea,
label,
navigator,
image {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

/* 去除按钮默认边框 */
button {
  background: none;
  border: none;
}

button::after {
  border: none;
}

/* 去除滚动条 */
::-webkit-scrollbar {
  display: none;
  width: 0;
  height: 0;
  color: transparent;
}

/* 去除长按高亮 */
view,
text,
navigator,
input {
  -webkit-tap-highlight-color: transparent;
}

/* 去除输入框默认样式 */
input {
  outline: none;
  border: none;
  list-style: none;
  background: none;
}

/* 图片默认样式 */
image {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
