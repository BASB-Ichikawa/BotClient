<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router';
import { PublicClientApplication } from "@azure/msal-browser"

const config = {
  auth: {
    //clientId: '02a46cb4-3c1e-46a0-b2c3-e8c5e6ac14b9', // SPA側のクライアントID
    //authority: 'https://login.microsoftonline.com/16b3c013-d300-468d-ac64-7eda0820b6d3',
    //authority: 'https://login.microsoftonline.com/organizations', // マルチテナント用の設定 (但し、MSが公式に許可しているサービス(GrapAPIなど)のAPIに限る)

    clientId: 'c982c769-156c-43a2-a976-38fd49b500f8', // SPA側のクライアントID
    authority: 'https://login.microsoftonline.com/c138f920-1c56-4cfa-b535-c025b2aedb44',
    redirectUri: import.meta.env.VITE_WEB_URL
  },
  cache: {
    cacheLocation: "sessionStorage",
    storeAuthStateInCookie: false,
  }
}

const router = useRouter();
let app: PublicClientApplication = new PublicClientApplication(config)

onMounted(async () => {
  const response = await app.handleRedirectPromise().catch(error => console.error({error}))
  // IDPからのリダイレクト時のみresponseに値が含まている (つまり画面の初期表示時はnull)
  if (response !== null) {
    // 複数のIDPで認証した場合は複数のAccountが取得される
    if (app.getAllAccounts().length > 0) {
      app.setActiveAccount(app.getAllAccounts()[0]);
      // アクセストークンを取得
      const result = await app.acquireTokenSilent({
        // APIを公開した時に発行されるスコープ値 (SPA側のSPに事前にアクセス許可を追加しておく必要あり)
        //scopes: ['api://c527d880-7610-4fd4-918e-bbbda5d7a2b2/WebApiUsingDirectLine'],
        scopes: ["User.Read"],
      })
      alert(result.accessToken)
      router.push('/top')
    }
  }

})

async function onClickLogin() {
  if (app.getAllAccounts().length > 0) {
    app.setActiveAccount(app.getAllAccounts()[0]);
    const result = await app.acquireTokenSilent({
      scopes: ['User.Read'],
    })
    router.push('/top')
  } else {
    // Redirect型ログイン
    await app.acquireTokenRedirect({
      redirectStartPage: location.href,
      scopes: ['User.Read']
    })

    // Popup型ログイン
    // const result = await app.acquireTokenPopup ({
    //   scopes: ['User.ReadWrite'],
    //   redirectUri: 'http://localhost:5173'
    // })    
  }
}

</script>

<template>
  <div>
    <button @click="onClickLogin">AADログイン</button>
  </div>
</template>
