<script setup lang="ts">
import { onMounted } from 'vue'
import { RouterLink, RouterView } from 'vue-router'
// import * as SpeechSDK from "microsoft-cognitiveservices-speech-sdk";
import axios from 'axios'
import { BlobServiceClient } from "@azure/storage-blob";


let recorder: any
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
    console.log(chunks)

    let data = new FormData();
    data.append('file', chunks[0], 'audio.wav');
    axios.post('https://app-bot-study.azurewebsites.net/api/Conversation/Upload', data, {
      headers: { 'content-type': 'multipart/form-data' }
    })

    // const blobServiceClient = new BlobServiceClient("https://stdllabpoc.blob.core.windows.net");
    // const containerClient = blobServiceClient.getContainerClient('audits');
    // const blockBlobClient = containerClient.getBlockBlobClient('audit.wav');
    // blockBlobClient.uploadData(new Blob(chunks));
  });
})

async function onStartRecording() {
  recorder.start();
};

async function onStopRecording() {
  recorder.stop();
}
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />
    <div class="wrapper">
      Speach To Text検証
      <button @click="onStartRecording">開始</button>
      <button @click="onStopRecording">停止</button>
    </div>
  </header>
  <RouterView />
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
