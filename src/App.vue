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

const CheckForCommand = (results) => {
  results.forEach(result => {
    if (result.isFinal) {
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
      } else if (t.includes('hello')) {
        alert('Hello there!')
      } else if (t.includes('open Google')) {
        window.open('https://www.google.com', '_blank')
      }
    }
  })
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

const clearTranscript = () => {
  transcript.value = ''
}

const copyTranscript = () => {
  navigator.clipboard.writeText(transcript.value)
}
</script>

<template>
  <div class="app">
    <h1>VocalFlow</h1>
    <button :class="{'mic-active': isRecording}" @click="ToggleMic">
      <i :class="{'fas fa-microphone': !isRecording, 'fas fa-microphone-alt-slash': isRecording}"></i>
      <span>{{ isRecording ? 'Stop Recording' : 'Start Recording' }}</span>
    </button>
    <div class="textarea-wrapper">
      <textarea class="transcript" v-model="transcript" readonly></textarea>
      <div class="icons">
        <i class="fas fa-copy" @click="copyTranscript"></i>
        <i class="fas fa-times" @click="clearTranscript"></i>
      </div>
    </div>
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
  background: #f5f5f5;
  color: #333;
}

.app {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
  display: grid;
}

h1 {
  margin-bottom: 20px;
  font-size: 2.5rem;
}

button {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
  margin-left: 6px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  border: none;
  border-radius: 5px;
  background-color: #007bff;
  color: white;
  transition: background-color 0.3s, transform 0.3s;
}

button .fas {
  margin-right: 8px;
  font-size: 1.2rem;
}

button.mic-active {
  background-color: #dc3545;
}

button:hover {
  background-color: #0056b3;
  transform: scale(1.05);
}

button.mic-active:hover {
  background-color: #c82333;
}

.textarea-wrapper {
  position: relative;
  margin-top: 20px;
}

.transcript {
  width: 100%;
  height: 300px;
  padding: 10px;
  font-size: 18px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background: #2b2b2b;
  resize: none;
  overflow: auto;
  white-space: pre-wrap;
  color: #ffffff;
  padding-right: 40px;
}

.icons {
  position: absolute;
  top: 10px;
  right: 10px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.icons .fas {
  font-size: 1.2rem;
  color: #ffffff;
  cursor: pointer;
}

@media (max-width: 600px) {
  .transcript {
    height: 200px;
  }

  h1 {
    font-size: 2rem;
  }

  button {
    font-size: 14px;
    padding: 8px 16px;
  }

  .icons .fas {
    font-size: 1rem;
  }
}
</style>
