<!-- eslint-disable no-unused-vars -->
<script setup>
import { ref, onMounted, reactive, onUpdated } from "vue";
import { storeToRefs } from "pinia";
// import Stomp from "webstomp-client";
import { CompatClient, Stomp } from "@stomp/stompjs";
import { useRoute, useRouter } from "vue-router";
import { listChatRoom, loadMessages } from "@/api/chat";
import { sendNotification } from "@/api/notification";
import { useMemberStore } from "@/stores/auth";

const memberStore = useMemberStore();
const { getUserInfo } = memberStore;

const route = useRoute();
const router = useRouter();

// Reactive data
const chats = ref([]);
const roomid = route.params.roomid;
const accessToken = localStorage.getItem("accessToken");
const message = ref(null);
const nickname = ref(null);
const savedChats = ref([]);
const loginUser = ref(null);

let socket = reactive(null);
let stomp = reactive(null);

onMounted(async () => {
  let token = localStorage.getItem("accessToken"); //현재 로그인한 유저
  const userInfo = await getUserInfo(token);
  loginUser.value = userInfo;
  nickname.value = userInfo.nickname;
  fetchAndLoadMessages(); //저장한 대화 가져오기
  connect();
  scrollToBottom();
});

onUpdated(() => {
  scrollToBottom();
});

const fetchAndLoadMessages = () => {
  const chatRoomId = roomid;
  loadMessages(
    chatRoomId,
    ({ data }) => {
      savedChats.value = data;
    },
    (error) => {
      console.error(error);
    }
  );
};

const scrollToBottom = () => {
  // Scroll to the bottom of the chat container
  const chatContainer = document.getElementById("chatContainer");
  chatContainer.scrollTop = chatContainer.scrollHeight;
};

// Methods
const sendMessage = async (e) => {
  if (message.value !== "") {
    send();
    message.value = "";
    scrollToBottom();
  }
  // Define the parameters for sendNotification
  const receiveId = 1;
  const type = 0;
  const notificationTitle = '알림 제목';
  const notificationBody = '알림 내용';

  // Call sendNotification function with the specified parameters
  await sendNotification(receiveId, type, notificationTitle, notificationBody);

};



const send = () => {

  const msg = {
    chatRoomId: roomid,
    sender: loginUser.value.nickname,
    message: message.value,
    messageType: "TALK",
  };

  stomp.send(
    "/pub/chat/message",
    { Authorization: `Bearer ${accessToken}` },
    JSON.stringify(msg)
  );
};

const connect = () => {
  const serverURL = "wss://i10c210.p.ssafy.io/ws-stomp"; //배포 시 서버주소
  // const serverURL = "ws://localhost:8080/ws-stomp";

  stomp = Stomp.client(serverURL);
  // stomp.reconnect_delay = 5000;
  stomp.connect(
    { Authorization: "Bearer " + accessToken },
    (frame) => {
      // 소켓 연결 성공

      stomp.subscribe("/sub/chat/room/" + roomid, (res) => {
        const chat = JSON.parse(res.body);
        chats.value.push({
          chatRoomId: chat.chatRoomId,
          message: chat.message,
          sender: chat.sender,
          messageType: chat.messageType,
        });
      });
    },
    (error) => {
      // 소켓 연결 실패
      console.log("소켓 연결 실패", error);
    }
  );
};

// Connect when the component is created
</script>

<template>
  <div>
    <!-- <button class="back-button btn" @click="goBack">
      <img src="@/assets/back.png" alt="" />
    </button> -->
    <div class="chat-container">
      <div class="chat-messages" ref="chatContainer" id="chatContainer">
        <div
          v-for="(schat, index) in savedChats"
          :key="index"
          :class="
            schat.sender === loginUser.nickname ? 'my-chat' : 'their-chat'
          "
        >
          <div class="message">{{ schat.message }}</div>
        </div>
        <div
          v-for="(chat, index) in chats"
          :key="index"
          :class="chat.sender === loginUser.nickname ? 'my-chat' : 'their-chat'"
        >
          <div class="message">{{ chat.message }}</div>
        </div>
      </div>
      <div class="chat-input">
        <textarea
          style="resize: none"
          v-model="message"
          placeholder="메시지 입력"
          @keydown.enter="sendMessage"
        ></textarea>
        <button class="input-button" @click="sendMessage">전송</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  border: 1px solid cornflowerblue;
  display: flex;
  flex-direction: column;
  height: 80vh; /* 높이를 조정할 부분입니다. */
  padding: 20px;
}
.chat-messages {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: auto;
}

.my-chat {
  align-self: flex-end;
  background-color: #0084ff;
  color: #fff;
  max-width: 70%;
  padding: 8px;
  margin: 8px 8px 8px 0;
  border-radius: 6px;
  text-align: right;
  position: relative;
}

.their-chat {
  align-self: flex-start;
  background-color: #f0f0f0;
  max-width: 70%;
  padding: 8px;
  margin: 8px 0;
  border-radius: 6px;
  text-align: left;
  color: #282828;
  position: relative;
}

.chat-input {
  display: flex;
  align-items: center;
  margin-top: 10px;
}

textarea {
  flex: 1;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-right: 10px;
  touch-action: manipulation;
  font-size: 16px;
}

.input-button {
  color: #fff;
  background-color: #0084ff;
  border: none;
  padding: 15px 10px;
  border-radius: 5px;
  cursor: pointer;
}

.back-button {
  border: none;
  padding: 8px 16px;
  border-radius: 5px;
  cursor: pointer;
  margin-bottom: 10px;
}
</style>
