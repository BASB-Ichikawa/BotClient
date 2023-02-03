<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { PublicClientApplication } from "@azure/msal-browser"

const config = {
  auth: {
    clientId: '18c62a00-926f-4275-8e03-65342c3051e1',
    authority: 'https://login.microsoftonline.com/aca5139d-3b2d-489e-b94b-1d7277819e3b',
    // redirectUri: "http://localhost:5173/top"
    redirectUri: 'https://proud-meadow-03aee4e00.2.azurestaticapps.net/top'
  },
  cache: {
    cacheLocation: "localStorage",
    storeAuthStateInCookie: false,
  }
}

let app: PublicClientApplication = new PublicClientApplication(config)

onMounted(async () => {
  app.handleRedirectPromise().then(async (response: any) => {
    if (response !== null) {
      if (app.getAllAccounts().length > 0) {
        app.setActiveAccount(app.getAllAccounts()[0]);
        const result = await app.acquireTokenSilent({
          scopes: ['User.Read'],
        })
        console.log(result.accessToken);
      }
    }
  })
})

async function onClickLogin() {
  if (app.getAllAccounts().length > 0) {
    app.setActiveAccount(app.getAllAccounts()[0]);
    const result = await app.acquireTokenSilent({
      scopes: ['User.Read'],
    })

    localStorage.setItem('token', result.accessToken)
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
