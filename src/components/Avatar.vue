<template>
  <img
    :src="computedSrc"
    :alt="alt"
    :class="['avatar-img', customClass]"
    :style="avatarStyle"
    @error="onError"
  />
</template>

<script setup>
import { ref, computed } from 'vue';
const props = defineProps({
  src: { type: String, default: '' },
  alt: { type: String, default: '頭像' },
  size: { type: [String, Number], default: 40 }, // px
  customClass: { type: String, default: '' },
})

// 狀態值
const errorFlag = ref(false)
const serverAvatar = ref('') // 🆕 從後端抓來的 avatar 圖檔名稱
const defaultAvatar = `${baseUrl}/images/User/default.png`

// 若父層沒傳 src，就從 API 取得個人資料
onMounted(async () => {
  if (!props.src) {
    try {
      const res = await axios.get(`${baseUrl}/api/UserProfile/profile`, {
        withCredentials: true,
      })
      serverAvatar.value = res.data.avatar || ''
      console.log('從後端取得頭像檔名:', serverAvatar.value)
    } catch (err) {
      console.error('取得頭像資料失敗:', err)
    }
  }
})

// 計算頭像來源網址
const computedSrc = computed(() => {
  const fileName = props.src?.trim() || serverAvatar.value?.trim()
  if (errorFlag.value || !fileName) {
    console.log('使用預設頭像，原因：', errorFlag.value ? '載入失敗' : '無來源圖片')
    return defaultAvatar
  }
  // 如果已經是完整網址，直接回傳
  if (fileName.startsWith('http://') || fileName.startsWith('https://')) {
    return fileName
  }
  const finalUrl = `${baseUrl}/images/User/${fileName}`
  console.log('嘗試載入圖片：', finalUrl)
  return finalUrl
})

// 樣式處理
const avatarStyle = computed(() => ({
  width: typeof props.size === 'number' ? props.size + 'px' : props.size,
  height: typeof props.size === 'number' ? props.size + 'px' : props.size,
  borderRadius: '50%',
  objectFit: 'cover',
}));
function onError() {
  errorFlag.value = true;
}
</script>

<style scoped>
.avatar-img {
  display: inline-block;
  border-radius: 50%;
  object-fit: cover;
  background: #f4f5f7;
}
</style>
