<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { RouterLink, RouterView } from 'vue-router'
// import * as SpeechSDK from "microsoft-cognitiveservices-speech-sdk";
import axios from 'axios'

let recorder: any
let fileNames = ref(<string[]>[])

onMounted(async () => {
  const stream = await navigator.mediaDevices.getUserMedia({
    audio: true,
    video: false
  })

  recorder = new MediaRecorder(stream, {
    mimeType: 'video/webm;codecs=vp9'
  });

  const chunks = <any>[];
  recorder.addEventListener('dataavailable', function (ele: any) {
    if (ele.data.size > 0) {
      chunks.push(ele.data);
    }
  });

  recorder.addEventListener('stop', function () {
    chunks.forEach(async (chunk: any, index: number) => {
      let data = new FormData();
      fileNames.value.push(`audio${index + 1}.wav`)
      data.append('file', chunk, fileNames.value[index]);
      await axios.post('https://app-bot-study.azurewebsites.net/api/Conversation/Upload', data, {
        headers: { 'content-type': 'multipart/form-data' }
        //headers: { 'content-type': 'audio/wav' }        
      })

      console.log(fileNames.value[index])
    });
  });
})

async function onStartRecording() {
  alert("録音開始")
  recorder.start();
};

async function onStopRecording() {
  alert("録音停止")
  recorder.stop();
}
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />

  </header>
  <div style="display: flex; flex-direction: column;">
    <p style="display: flex; justify-content: center">Speach To Text検証</p>
    <div style="display: flex; justify-content: center; margin-top: 10px;">
      <button @click="onStartRecording" style="margin-right: 10px">開始</button>
      <button @click="onStopRecording">停止</button>
    </div>
    <div>
      <ul>
        <li v-for="name in fileNames">
          <a :href="'https://stdllabpoc.blob.core.windows.net/audits/' + name">{{name}}</a>
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
