<script setup lang="ts">
import { onMounted, ref } from 'vue'
// import * as SpeechSDK from "microsoft-cognitiveservices-speech-sdk";
import axios from 'axios'
import { PublicClientApplication, type AuthenticationResult } from "@azure/msal-browser"

let recorder: any
let originalFileNames = ref(<string[]>[])
let convertedFileNames = ref(<string[]>[])
let recognizedContent = ref('<認識済みテキストを表示>')
let accessToken: string | null

const config = {
  auth: {
    clientId: '18c62a00-926f-4275-8e03-65342c3051e1', // SPA側のクライアントID (MSDNテナント)
    authority: 'https://login.microsoftonline.com/aca5139d-3b2d-489e-b94b-1d7277819e3b', // MSDNテナント    
  }
}

onMounted(async () => {
  var app = new PublicClientApplication(config)
  let result = await app.acquireTokenSilent({
    scopes: ['api://c527d880-7610-4fd4-918e-bbbda5d7a2b2/WebApiUsingDirectLine'],
  });
  accessToken = result.accessToken
  console.log(accessToken);

  const stream = await navigator.mediaDevices.getUserMedia({
    audio: true,
    video: false
  })

  recorder = new MediaRecorder(stream, {
    //mimeType: 'video/webm;codecs=vp9'
    mimeType: 'audio/webm'
  });

  const chunks = <any>[];
  recorder.addEventListener('dataavailable', function (element: any) {
    if (element.data.size > 0) {
      chunks.push(element.data);
    }
  });

  recorder.addEventListener('stop', function () {
    chunks.forEach(async (chunk: any, index: number) => {
      let data = new FormData();
      originalFileNames.value.push(`audio${index + 1}.webm`)
      data.append('file', chunk, originalFileNames.value[index]);
      const result = await axios.post(`${import.meta.env.VITE_API_URL}/api/Conversation/Upload`, data, {
        headers: {
          Authorization: `Bearer ${accessToken}`,
          'Content-Type': 'multipart/form-data'
        }
      })

      convertedFileNames.value.push(result.data.fileName)
      if (recognizedContent.value === '認識中...') {
        recognizedContent.value = '';
      }
      recognizedContent.value += result.data.recognizedText;
    });
  });
})

async function onStartRecording() {
  convertedFileNames.value.length = 0;
  recognizedContent.value = '録音中...';
  recorder.start();
};

async function onStopRecording() {
  recognizedContent.value = '認識中...';
  recorder.stop();
}

async function onConfirmAuth() {
  const result = await axios.post(`${import.meta.env.VITE_API_URL}/api/Conversation/CheckAuth`, {}, {
    headers: {
      Authorization: `Bearer ${accessToken}`
    }
  })

  alert(`ようこそ ${result.data}さん`)
}

async function onExecuteBatch() {
  const result = await axios.post(`${import.meta.env.VITE_API_URL}/api/Conversation/Batch`, {}, {
    headers: {
      Authorization: `Bearer ${accessToken}`
    }
  })

  alert(`${result.data}`)
}


</script>

<template>
  <section>
    <p style="display: flex; justify-content: center">【Speech To Text検証】</p>
    <div class="section_action">
      <button @click="onStartRecording" style="margin-right: 10px">録音開始</button>
      <button @click="onStopRecording">録音停止 -> 認識開始</button>
    </div>
    <div class="section">
      <p class="section_title">■ BLOBに保存した音声データ</p>
      <ul>
        <li v-for="name in convertedFileNames">
          <a :href="'https://stdllabpoc.blob.core.windows.net/audio/' + name">{{ name }}</a>
        </li>
      </ul>
    </div>
    <div class="section">
      <p class="section_title">■ STT認識結果</p>
      <p>{{ recognizedContent }}</p>
    </div>
    <div class="section">
      <p>アクセス許可確認</p>
      <button @click="onConfirmAuth" style="margin-right: 10px">確認</button>
    </div>
    <div class="section">
      <p>バッチ文字起こし</p>
      <button @click="onExecuteBatch" style="margin-right: 10px">実行</button>
    </div>
  </section>
</template>

<style scoped>
.section {
  margin-top: 10px;
}

.section_action {
  margin-top: 10px;
  display: flex;
  justify-content: center;
}

.section_title {
  margin-top: 10px;
  display: flex;
  justify-content: start;
}
</style>
