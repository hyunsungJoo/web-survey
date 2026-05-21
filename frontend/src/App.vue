<template>
  <div class="survey-wrapper">
    
    <Transition name="fade">
      <div v-if="showToast" class="toast-notification">
        ✅ 소중한 의견이 기록되었습니다!
      </div>
    </Transition>

    <div 
      class="survey-container" 
      :style="{ transform: `scale(${containerScale})`, transformOrigin: 'center center' }"
    >
      <div class="survey-header">      
        <h1 class="main-title">오늘 식사 맛있게 드셨나요?</h1>
        <h2 class="sub-title-1">고객님의 소중한 의견을 들려주세요.</h2>
      </div>
      
      <div class="rating-group" :class="{ 'is-loading': loading }">
        <div 
          v-for="item in ratingOptions" 
          :key="item.score"
          :class="['rating-item', { active: selectedOption.score === item.score }]"
          @click="submitRating(item)"
        >
          
          <!-- ✅ Microsoft Fluent Emoji SVG -->
          <div class="emoji">
            <img :src="item.emoji" :alt="item.text" />
          </div>

          <div class="content-box">
            <div class="text">{{ item.text }}</div>
            <div class="score">{{ item.score }}점</div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import axios from 'axios'

/*
  ✅ public/emoji 폴더에 SVG 넣기

  public/
   └── emoji/
       ├── very-bad.svg
       ├── bad.svg
       ├── normal.svg
       ├── good.svg
       └── very-good.svg
*/

const ratingOptions = [
  { 
    score: 1, 
    text: '매우 불만족', 
    emoji: '/emoji/very-bad.svg'
  },
  { 
    score: 2, 
    text: '불만족', 
    emoji: '/emoji/bad.svg'
  },
  { 
    score: 3, 
    text: '보통', 
    emoji: '/emoji/normal.svg'
  },
  { 
    score: 4, 
    text: '만족', 
    emoji: '/emoji/good.svg'
  },
  { 
    score: 5, 
    text: '매우 만족', 
    emoji: '/emoji/very-good.svg'
  }
]

const selectedOption = ref({ score: null, text: '' })
const loading = ref(false)
const showToast = ref(false)

const GOOGLE_WEB_APP_URL = 'https://script.google.com/macros/s/AKfycbwybid-i3loF0bbZsqFWl9ZXcYK9gGLD2KVnXFoNWahcY1Pombp3OAGepdmqIUB9qrH/exec'

const containerScale = ref(1)

const updateScale = () => {

  // 모바일은 스케일 제거
  if (window.innerWidth <= 500) {
    containerScale.value = 1
    return
  }
  
  const baseWidth = 1750 
  const baseHeight = 960 
  
  const availableWidth = window.innerWidth - 60
  const availableHeight = window.innerHeight - 60
  
  const scaleX = availableWidth / baseWidth
  const scaleY = availableHeight / baseHeight
  
  containerScale.value = Math.min(scaleX, scaleY, 1)
}

onMounted(() => {
  updateScale()
  window.addEventListener('resize', updateScale)
})

onUnmounted(() => {
  window.removeEventListener('resize', updateScale)
})

const submitRating = (item) => {

  if (loading.value) return 
  
  selectedOption.value = { 
    score: item.score, 
    text: item.text 
  }

  loading.value = true
  showToast.value = true

  const payload = {
    score: item.score,
    satisfaction: item.text
  }

  axios.post(
    GOOGLE_WEB_APP_URL, 
    JSON.stringify(payload),
    {
      headers: {
        'Content-Type': 'text/plain'
      }
    }
  ).catch((error) => {
    console.error('백그라운드 전송 실패:', error)
  })
  
  setTimeout(() => {
    showToast.value = false
    selectedOption.value = { score: null, text: '' }
    loading.value = false
  }, 1000)
}
</script>

<style scoped>

/* =========================
   공통 레이아웃
========================= */

.survey-wrapper {
  background-color: #444444;
  height: 100vh;
  height: 100dvh;
  width: 100vw;

  display: flex;
  justify-content: center;
  align-items: center;

  padding: 40px;
  box-sizing: border-box;

  overflow: hidden;
}

@media (min-width: 501px) {

  .survey-container {
    width: 1750px;
    flex-shrink: 0;
  }
}

.survey-container {
  text-align: center;
  font-family: sans-serif;
  position: relative;
  background-color: transparent;
}

.survey-header {
  margin-bottom: 100px;
}

/* =========================
   제목
========================= */

.main-title {
  font-size: 6rem;
  font-weight: 800;
  color: #ffffff;

  margin: 0 0 15px 0;
  letter-spacing: -1.5px;
}

.sub-title-1 {
  font-size: 3.8rem;
  font-weight: 600;
  color: #ffffff;

  margin: 10px 0 28px 0;
}

/* =========================
   설문 카드
========================= */

.rating-group {

  display: flex;
  justify-content: space-between;

  gap: 45px;

  margin-bottom: 40px;

  transition: opacity 0.2s ease;
}

.rating-group.is-loading {
  pointer-events: none;
  opacity: 0.6;
}

.rating-item {

  flex: 1;

  border: 4px solid #333333;
  border-radius: 32px;

  padding: 75px 25px;

  cursor: pointer;

  transition: all 0.25s ease;

  background-color: #ffffff;
  color: #1a202c;

  box-shadow: 0 12px 35px rgba(0, 0, 0, 0.25);

  -webkit-tap-highlight-color: transparent;
  outline: none;
}

@media (hover: hover) {

  .rating-item:hover {
    border-color: #42b883;
    background-color: #f4fbf8;

    transform: translateY(-12px);
  }
}

.rating-item.active {

  border-color: #42b883;
  background-color: #42b883;

  color: #ffffff;
}

/* =========================
   이모지 이미지
========================= */

.emoji {

  display: flex;
  justify-content: center;
  align-items: center;

  margin-bottom: 30px;
}

.emoji img {

  width: 150px;
  height: 150px;

  object-fit: contain;

  user-select: none;
  pointer-events: none;
}

/* =========================
   텍스트
========================= */

.content-box {

  display: flex;
  flex-direction: column;
}

.text {

  font-size: 2.2rem;
  font-weight: bold;

  margin-bottom: 12px;
}

.score {

  font-size: 1.7rem;
  opacity: 0.7;
}

.rating-item.active .score {
  opacity: 0.9;
}

/* =========================
   모바일
========================= */

@media (max-width: 500px) {

  .survey-wrapper {

    display: block;

    padding: 40px 20px;

    height: 100vh;
    height: 100dvh;

    overflow-y: auto;
  }

  .survey-container {

    width: 100%;
    transform: none !important;
  }

  .survey-header {
    margin-bottom: 40px;
  }

  .main-title {

    font-size: 2.5rem;
    margin-bottom: 8px;
  }

  .sub-title-1 {

    font-size: 1.6rem;
    margin-bottom: 12px;
  }

  .rating-group {

    flex-direction: column;
    gap: 16px;

    width: 100%;
  }

  .rating-item {

    padding: 25px 30px;

    display: flex;
    align-items: center;
    justify-content: flex-start;

    gap: 25px;

    text-align: left;

    border-radius: 16px;
  }

  .rating-item:active {
    background-color: #f9f9f9;
  }

  .emoji {

    margin-bottom: 0;
  }

  .emoji img {

    width: 80px;
    height: 80px;
  }

  .content-box {

    flex-direction: row;
    align-items: center;

    gap: 12px;

    width: 100%;
  }

  .text {

    font-size: 1.4rem;
    margin-bottom: 0;
  }

  .score {

    font-size: 1.2rem;
    margin-left: auto;
  }

  .toast-notification {

    top: 30px !important;

    padding: 18px 40px !important;

    font-size: 1.4rem !important;

    border-radius: 50px !important;

    border-width: 3px !important;
  }
}

/* =========================
   토스트
========================= */

.toast-notification {

  position: fixed;

  top: 80px;
  left: 50%;

  transform: translateX(-50%);

  background-color: #222222;
  color: #ffffff;

  padding: 30px 80px;

  border-radius: 100px;

  font-size: 2.5rem;
  font-weight: 800;

  box-shadow: 0 15px 50px rgba(0, 0, 0, 0.6);

  z-index: 99999;

  white-space: nowrap;

  border: 4px solid #42b883;
}

/* =========================
   애니메이션
========================= */

.fade-enter-active,
.fade-leave-active {

  transition: opacity 0.3s ease, transform 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {

  opacity: 0;
  transform: translate(-50%, -30px);
}

</style>

<style>

html,
body {

  margin: 0 !important;
  padding: 0 !important;

  height: 100% !important;
  width: 100% !important;

  background-color: #444444 !important;

  overflow: hidden !important;
}

</style>