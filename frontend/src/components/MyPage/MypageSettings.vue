<template>
    <div>
        <button class="settings-back-button btn" @click="goBack">
          <img src="@/assets/back.png" alt="">
        </button>
        <div class="settings">
          <router-link to="/userInfoEdit" class="nav-link">
              개인정보 변경
            </router-link>
            <router-link to="/subscribe" class="nav-link">
              알림 제어
            </router-link>
            <router-link to="/common" class="nav-link">
              일반 설정
            </router-link>
            <router-link to="/appinfo" class="nav-link">
              앱 정보
            </router-link>
            <button class="logout-button btn" @click="confirmLogout">로그아웃</button>
            <div v-if="showLogoutConfirmation" class="modal">
              <div class="modal-content">
                <p>정말 로그아웃 하시겠습니까?</p>
                <button class="confirm-button btn" @click="logoutConfirmed">확인</button>
                <button class="cancel-button btn" @click="cancelLogout">취소</button>
              </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import router from '@/router';
import { ref } from 'vue'
// import { useMemberStore } from '@/stores/auth'

// const memberStore = useMemberStore();
// const { userLogout, getUserInfo } = memberStore;

function goBack() {
  window.history.back()
}

const showLogoutConfirmation = ref(false);

function confirmLogout() {
  showLogoutConfirmation.value = true;
}

function cancelLogout() {
  showLogoutConfirmation.value = false;
}

function logoutConfirmed() {
  localStorage.removeItem("accessToken");
  showLogoutConfirmation.value = false;
  router.push('/login').then(() => {
    window.location.reload(); // 페이지 새로고침
  });
}
</script>

<style scoped>
.settings-back-button {
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-right: auto;
  display: flex;
}

.settings {
  font-size: 25px;
  padding: 20px;
  margin-left: 10px;
}

.logout-button {
  margin-top: 20px;
  background-color: red;
  color: white;
}

.modal {
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
}

.confirm-button, .cancel-button {
  margin: 10px;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.confirm-button {
  background-color: green;
  color: white;
}

.cancel-button {
  background-color: grey;
  color: white;
}
</style>