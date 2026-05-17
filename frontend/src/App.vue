<template>
  <div class="survey-wrapper">
    <div class="survey-container">
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
import { ref } from 'vue'
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

const submitRating = async (item) => {
  if (loading.value) return 
  
  selectedOption.value = { score: item.score, text: item.text }
  loading.value = true

  try {
    const payload = {
      score: item.score,
      satisfaction: item.text
    }

    const response = await axios.post(GOOGLE_WEB_APP_URL, JSON.stringify(payload), {
      headers: {
        'Content-Type': 'text/plain'
      }
    })
    
    if (response.data.result === 'success') {
      showToast.value = true
      setTimeout(() => {
        showToast.value = false
        selectedOption.value = { score: null, text: '' }
      }, 1000)
    }
  } catch (error) {
    console.error('제출 에러:', error)
    alert('제출에 실패했습니다. 다시 시도해 주세요.')
    selectedOption.value = { score: null, text: '' }
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
/* 1. 모니터 전체 화면 스타일 */
.survey-wrapper {
  background-color: #444444;
  min-height: 100vh;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 80px 40px; 
  box-sizing: border-box;
}

/* 초초대형 스크린을 위한 가로폭 확장 */
.survey-container {
  width: 100%;
  max-width: 1750px; 
  text-align: center;
  font-family: sans-serif;
  position: relative;
  background-color: transparent;
}

/* 타이틀 영역 스타일링 (초초대형 스케일) */
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

/* 만족도 카드 스타일 (자이언트 스케일) */
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
}

.rating-item:hover {
  border-color: #42b883;
  background-color: #f4fbf8;
  transform: translateY(-12px); 
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
@media (max-width: 1200px) {
  .main-title { font-size: 4rem; }
  .sub-title-1 { font-size: 2.6rem; }
  .sub-title-2 { font-size: 1.8rem; }
  .emoji { font-size: 6.5rem; }
  .text { font-size: 1.6rem; }
  .score { font-size: 1.2rem; }
}

@media (max-width: 768px) {
  .survey-wrapper {
    padding: 30px 15px;
  }

  .main-title { font-size: 2.5rem; margin-bottom: 10px; }
  .sub-title-1 { font-size: 1.6rem; margin-bottom: 12px; }
  .sub-title-2 { font-size: 1.1rem; }

  .rating-group {
    flex-direction: column;
    gap: 16px;
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

  .rating-item:hover {
    transform: translateX(6px);
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
  background-color: #444444 !important;
}
</style>