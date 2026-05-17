<template>
  <div class="survey-wrapper">
    <div 
      class="survey-container" 
      :style="{ transform: `scale(${containerScale})`, transformOrigin: 'center center' }"
    >
      <div class="survey-header">
        <h1 class="main-title">고객님</h1>
        <h2 class="sub-title-1">오늘 식사 맛있게 드셨나요?</h2>
        <p class="sub-title-2">고객님의 소중한 의견을 들려주세요.</p>
      </div>
      
      <div class="rating-group" :class="{ 'is-loading': loading }">
        <div 
          v-for="item in ratingOptions" 
          :key="item.score"
          :class="['rating-item', { active: selectedOption.score === item.score }]"
          @click="submitRating(item)"
        >
          <div class="emoji">{{ item.emoji }}</div>
          <div class="content-box">
            <div class="text">{{ item.text }}</div>
            <div class="score">{{ item.score }}점</div>
          </div>
        </div>
      </div>

      <Transition name="fade">
        <div v-if="showToast" class="toast-notification">
          ✅ 소중한 의견이 기록되었습니다!
        </div>
      </Transition>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import axios from 'axios'

const ratingOptions = [
  { score: 5, text: '매우 만족', emoji: '😍' },
  { score: 4, text: '만족', emoji: '😊' },
  { score: 3, text: '보통', emoji: '😐' },
  { score: 2, text: '불만족', emoji: '😨' },
  { score: 1, text: '매우 불만족', emoji: '😰' }
]

const selectedOption = ref({ score: null, text: '' })
const loading = ref(false)
const showToast = ref(false)

const GOOGLE_WEB_APP_URL = 'https://script.google.com/macros/s/AKfycbwybid-i3loF0bbZsqFWl9ZXcYK9gGLD2KVnXFoNWahcY1Pombp3OAGepdmqIUB9qrH/exec'

const containerScale = ref(1)

const updateScale = () => {
  if (window.innerWidth <= 768) {
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

// 🌟 체감 속도를 0초로 만드는 즉시 제출 함수
const submitRating = (item) => {
  if (loading.value) return 
  
  // 1. 시각적 피드백 즉시 반영
  selectedOption.value = { score: item.score, text: item.text }
  loading.value = true

  // 2. 💡 구글 서버의 응답을 기다리지 않고 "즉시 완료 토스트" 노출
  showToast.value = true

  // 3. 백엔드 데이터 전송은 뒷단(Background)에서 비동기로 조용히 실행
  const payload = {
    score: item.score,
    satisfaction: item.text
  }

  axios.post(GOOGLE_WEB_APP_URL, JSON.stringify(payload), {
    headers: {
      'Content-Type': 'text/plain'
    }
  }).catch((error) => {
    // 사용자는 이미 화면을 마친 상태이므로, 에러는 콘솔 로그에만 남겨 관리자가 확인하도록 합니다.
    console.error('백그라운드 전송 실패:', error)
  })
  
  // 4. 💡 구글 응답과 관계없이 1초 뒤에 화면을 깨끗하게 비워 다음 사람이 바로 참여하게 함
  setTimeout(() => {
    showToast.value = false
    selectedOption.value = { score: null, text: '' }
    loading.value = false
  }, 1000)
}
</script>

<style scoped>
/* 1. 모니터 전체 화면 스타일 */
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

@media (min-width: 769px) {
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
.main-title {
  font-size: 6rem; 
  font-weight: 800;
  color: #ffffff;
  margin: 0 0 24px 0;
  letter-spacing: -1.5px;
}
.sub-title-1 {
  font-size: 3.8rem; 
  font-weight: 600;
  color: #ffffff;
  margin: 0 0 28px 0;
}
.sub-title-2 {
  font-size: 2.4rem; 
  font-weight: 700;
  color: #f3f4f6;
  margin: 0;
}

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

/* 만족도 카드 스타일 */
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

.emoji {
  font-size: 9rem; 
  margin-bottom: 30px;
}

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

/* 2. 📱 모바일/태블릿용 반응형 스타일 */
@media (max-width: 768px) {
  .survey-wrapper {
    padding: 30px 15px;
    overflow-y: auto; 
  }

  .main-title { font-size: 2.5rem; margin-bottom: 10px; }
  .sub-title-1 { font-size: 1.6rem; margin-bottom: 12px; }
  .sub-title-2 { font-size: 1.1rem; }

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
    font-size: 3.5rem;
    margin-bottom: 0;
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
}

/* 대형 토스트 알림창 스타일 */
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
  z-index: 9999;
  white-space: nowrap;
  border: 4px solid #42b883;
}

/* Vue Transition 애니메이션 */
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
html, body {
  margin: 0 !important;
  padding: 0 !important;
  height: 100% !important;
  width: 100% !important;
  background-color: #444444 !important;
  overflow: hidden !important; 
}
</style>