<script setup>
import { ref, onMounted } from 'vue'

const transcript = ref('')
const isRecording = ref(false)

let Recognition = window.SpeechRecognition || window.webkitSpeechRecognition
let sr

const initializeRecognition = () => {
  if (Recognition) {
    sr = new Recognition()

    sr.continuous = true
    sr.interimResults = true

    sr.onstart = () => {
      console.log('Record Started')
      isRecording.value = true
    }

    sr.onend = () => {
      console.log('Record Stopped')
      isRecording.value = false
    }

    sr.onresult = (evt) => {
      for (let i = 0; i < evt.results.length; i++) {
        const result = evt.results[i]

        if (result.isFinal) {
          CheckForCommand(result)
        }
      }

      const t = Array.from(evt.results)
        .map(result => result[0])
        .map(result => result.transcript)
        .join('')

      transcript.value = t
    }
  } else {
    console.error('SpeechRecognition API is not supported in this browser.')
  }
}

onMounted(() => {
  initializeRecognition()
})

const CheckForCommand = (result) => {
  const t = result[0].transcript
  if (t.includes('stop recording')) {
    sr.stop()
  } else if (
    t.includes('what is the time') ||
    t.includes('what\'s the time')
  ) {
    sr.stop()
    alert(new Date().toLocaleTimeString())
    setTimeout(() => {
      if (sr) {
        sr.start()
      }
    }, 100)
  }
}

const ToggleMic = () => {
  if (!sr) {
    console.error('SpeechRecognition instance is not initialized.')
    return
  }

  if (isRecording.value) {
    sr.stop()
  } else {
    sr.start()
  }
}
</script>

<template>
  <div class="app">
    <h1>VocalFlow</h1>
    <button :class="{'mic-active': isRecording}" @click="ToggleMic">Record</button>
    <div class="transcript">{{ transcript }}</div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Courier New', Courier, monospace;
}

body {
  background: #ffffff;
  color: #030000;
}

.app {
  padding: 20px;
  text-align: center;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}

button.mic-active {
  background-color: red;
  color: white;
}

.transcript {
  margin-top: 20px;
  font-size: 18px;
  white-space: pre-wrap;
}
</style>
