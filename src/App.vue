<script setup>
import AgoraRTM from 'agora-rtm-sdk';
import { v4 as uuidv4 } from 'uuid';
import { ref, onMounted, nextTick } from 'vue';

const APP_ID = '7aa56656b07b4f93a2959621d89bb966';
const CHANNEL = 'testchat';
const client = AgoraRTM.createInstance(APP_ID);
const uid = uuidv4();
let channel;

const text = ref('');
const messages = ref([]);

const appendMessage = async (message) => {
  messages.value.push(message);
  await nextTick();
};
onMounted(async () => {
  await client.login({ uid, token: null });
  channel = await client.createChannel(CHANNEL);
  await channel.join();
  channel.on('ChannelMessage', (message, peerId) => {
    appendMessage({
      text: message.text,
      uid: peerId,
    });
  });
});
function sendMessage() {
  if (text.value === '') return;
  channel.sendMessage({ text: text.value, type: 'text' });
  appendMessage({
    text: text.value,
    uid,
  });
  text.value = '';
}
</script>

<template>
  <div class="panel">
        <div class="messages">
            <div class="inner">
              <div 
                  v-for="(message, index) in messages" 
                  :key="index" 
                  class="message"
                >
                  <div v-if="message.uid === uid" class="user-self">
                    <span class="you">You:</span> 
                    <span class="text">{{ message.text }}</span>
                  </div>
                  <div v-else class="friend">
                    <span class="him">Friend:</span>  
                    <span class="text">{{ message.text }}</span>
                  </div>
              </div>
            </div>

          </div>
          <form @submit.prevent="sendMessage">
              <input v-model="text" />
              <svg width="40px" height="40px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" @click="sendMessage">
                <g stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g>
                <g> <path fill-rule="evenodd" clip-rule="evenodd" d="M12 2C6.47715 2 2 6.47715 2 12C2 17.5228 6.47715 22 12 22C17.5228 22 22 17.5228 22 12C22 6.47715 17.5228 2 12 2ZM11.2929 7.29289C11.6834 6.90237 12.3166 6.90237 12.7071 7.29289L16.7071 11.2929C16.8946 11.4804 17 11.7348 17 12C17 12.2652 16.8946 12.5196 16.7071 12.7071L12.7071 16.7071C12.3166 17.0976 11.6834 17.0976 11.2929 16.7071C10.9024 16.3166 10.9024 15.6834 11.2929 15.2929L13.5858 13H8C7.44772 13 7 12.5523 7 12C7 11.4477 7.44772 11 8 11H13.5858L11.2929 8.70711C10.9024 8.31658 10.9024 7.68342 11.2929 7.29289Z" fill="#f0f0f0"></path></g>
              </svg>
          </form>
    </div>
</template>

