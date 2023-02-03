<script setup lang="ts">
import { onMounted, ref } from 'vue'
// import * as SpeechSDK from "microsoft-cognitiveservices-speech-sdk";
import axios from 'axios'

let recorder: any
let originalFileNames = ref(<string[]>[])
let convertedFileNames = ref(<string[]>[])
let recognizedContent = ref('')

onMounted(async () => {
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
      const result = await axios.post('https://app-bot-study.azurewebsites.net/api/Conversation/Upload', data, {
        headers: { 'content-type': 'multipart/form-data' }
      })

      convertedFileNames.value.push(result.data.fileName)
      recognizedContent += result.data.recognizedText;
      alert(recognizedContent)
    });    
  });
})

async function onStartRecording() {
  alert("録音開始")
  recorder.start();
};

async function onStopRecording() {
  recorder.stop();
}
</script>

<template>
  <section>
    <p style="display: flex; justify-content: center">【Speach To Text検証】</p>
    <div class="section_action">
      <button @click="onStartRecording" style="margin-right: 10px">録音開始</button>
      <button @click="onStopRecording">録音停止 -> 認識開始</button>
    </div>
    <div class="section_content">
      <p>■ BLOBに保存した音声データ</p>
      <ul>
        <li v-for="name in recognizedContent">
          <a :href="'https://stdllabpoc.blob.core.windows.net/audio/' + name">{{ name }}</a>
        </li>
      </ul>
    </div>
    <div class="section_content">
      <p>■ STT認識結果</p>
      {{ recognizedContent }}
    </div>
  </section>
</template>

<style scoped>
.section_action {
  margin-top: 10px;
  display: flex;
  justify-content: center;
}
.section_content {
  margin-top: 10px;
  display: flex;
  justify-content: start;
}
</style>
