<template>
  <div>

    <template v-if="invalidPath">
      <div>
        <h1>Enter a channel and username in the URL</h1>
        <p>Example: <a href="/xqc/xqc">/channel/username</a></p>
      </div>
    </template>

    <template v-else>
      <div id="title">
        <h1>Channel: {{ channel }}</h1>
        <h1>Username: {{ username }}</h1>
      </div>
  
      <div id="messages">
        <h2>Messages</h2>
        <ol>
          <li v-for="(message, index) in getMessages" :key="index">
            {{ message.ts }}: {{ message.text }}
          </li>
        </ol>
      </div>
  
      <div class="status-bar">
        <p>Client: {{ clientStatus }}</p>
      </div>
    </template>

  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useTitle } from '@vueuse/core'
import tmi from 'tmi.js'

const channel = ref(window.location.pathname.split('/')[1])
const username = ref(window.location.pathname.split('/')[2])
const messages = ref([])
const clientStatus = ref('disconnected')

const title = useTitle()

title.value= `${title.value} | ${channel.value}/${username.value}`

const client = tmi.Client({
  connection: {
    secure: true,
    reconnect: true
  },
  channels: [channel.value]
})

client.connect()

client.on('connected', () => {
  clientStatus.value = 'connected'
})

client.on('disconnected', () => {
  clientStatus.value = 'disconnected'
})

client.on('message', (channel, tags, message, self) => {
  if (self) return

  if (tags.username === username.value) {
    addMessage(message)
  }
})

const addMessage = (text) => { messages.value.push({ text, ts: new Date() }) }

const getMessages = computed(() => messages.value.reverse())

const invalidPath = computed(() => window.location.pathname.split('/').length !== 3)

</script>

<style scoped>

ol {
  list-style-type: none;
  padding: 0;
  margin: 0;
  max-height: 500px;
  overflow-y: scroll;
}

li:nth-child(even) {
  background-color: #313131;
}

#messages {
  margin-top: 20px;
}

#title {
  display: flex;
  justify-content: space-between;
}

.status-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #313131;
  color: #fff;
  padding: 10px;
  text-align: center;
}
</style>