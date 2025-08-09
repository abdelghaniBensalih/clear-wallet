<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Welcome to MagicCraft Wallet</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content class="onboarding-content">
      <div class="onboarding-card">
        <h2>Get Started</h2>
        <ion-button expand="block" @click="mode = 'create'"
          >Create New Wallet</ion-button
        >
        <ion-button expand="block" @click="mode = 'import'"
          >Import Existing Wallet</ion-button
        >
      </div>
      <div v-if="mode === 'create'" class="onboarding-card">
        <h3>Your 12-word Recovery Phrase</h3>
        <div class="mnemonic-box">{{ mnemonic }}</div>
        <ion-button expand="block" @click="mode = 'setPass'"
          >I have saved my phrase</ion-button
        >
      </div>
      <div v-if="mode === 'import'" class="onboarding-card">
        <h3>Enter Your 12-word Recovery Phrase</h3>
          <ion-textarea
            v-model="importMnemonic"
            :rows="3"
          placeholder="Enter mnemonic..."
        ></ion-textarea>
        <ion-button expand="block" @click="validateImport">Continue</ion-button>
      </div>
      <div v-if="mode === 'setPass'" class="onboarding-card">
        <h3>Set a Passphrase</h3>
        <ion-input
          v-model="passphrase"
          type="password"
          placeholder="Passphrase"
        ></ion-input>
        <ion-input
          v-model="passphrase2"
          type="password"
          placeholder="Repeat Passphrase"
        ></ion-input>
        <ion-button expand="block" @click="finishOnboarding">Finish</ion-button>
      </div>
      <ion-alert
        :is-open="alertOpen"
        header="Error"
        :message="alertMsg"
        :buttons="['OK']"
        @didDismiss="alertOpen = false"
      ></ion-alert>
    </ion-content>
  </ion-page>
</template>

<script lang="ts" setup>
import { ref, onMounted } from "vue";
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
  IonTextarea,
  IonInput,
  IonAlert,
} from "@ionic/vue";
import { ethers } from "ethers";
import { encrypt, getCryptoParams } from "@/utils/webCrypto";
import { saveAccount, saveSelectedAccount } from "@/utils/platform";
import router from "@/router";

const mode = ref("");
const mnemonic = ref("");
const importMnemonic = ref("");
const passphrase = ref("");
const passphrase2 = ref("");
const alertOpen = ref(false);
const alertMsg = ref("");

onMounted(() => {
  if (!mnemonic.value) {
    mnemonic.value = ethers.Wallet.createRandom().mnemonic?.phrase || "";
  }
});

function validateImport() {
  const words = importMnemonic.value.trim().split(/\s+/);
  if (words.length !== 12 && words.length !== 24) {
    alertMsg.value = "Mnemonic must be 12 or 24 words.";
    alertOpen.value = true;
    return;
  }
  mnemonic.value = importMnemonic.value.trim();
  mode.value = "setPass";
}

async function finishOnboarding() {
  if (!passphrase.value || passphrase.value !== passphrase2.value) {
    alertMsg.value = "Passphrases do not match.";
    alertOpen.value = true;
    return;
  }
  const wallet = ethers.Wallet.fromPhrase(mnemonic.value.trim());
  const cryptoParams = await getCryptoParams(passphrase.value);
  const encPk = await encrypt(wallet.privateKey, cryptoParams);
  await saveAccount({
    address: wallet.address,
    name: "Main",
    pk: wallet.privateKey,
    encPk,
  });
  await saveSelectedAccount({
    address: wallet.address,
    name: "Main",
    pk: wallet.privateKey,
    encPk,
  });
  router.push("/tabs/home");
}
</script>

<style scoped>
.onboarding-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  min-height: 100vh;
  background: #1a1b2f;
  color: #f1c40f;
}
.onboarding-card {
  background: #23244a;
  border-radius: 16px;
  box-shadow: 0 4px 24px #6e4b9e44;
  padding: 2rem;
  margin: 2rem 0;
  width: 90%;
  max-width: 400px;
  color: #f1c40f;
  text-align: center;
}
.mnemonic-box {
  background: #1a1b2f;
  color: #6e4b9e;
  border: 2px solid #f1c40f;
  border-radius: 8px;
  padding: 1rem;
  margin: 1rem 0;
  font-size: 1.1rem;
  word-break: break-word;
}
ion-button {
  --background: linear-gradient(90deg, #6e4b9e 0%, #f1c40f 100%);
  --color: #1a1b2f;
  margin-top: 1rem;
}
</style>
