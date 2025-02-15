<template>
  <div id="app">
    <header>
      <div class="header-content">
        <img
          :class="{ 'dark-png': store.state.theme === Theme.DARK }"
          class="logo"
          src="@/assets/logo-banner.png"
          alt="ChatALL"
        />
        <div class="column-icons">
          <img
            v-for="columnCount in 3"
            :id="`column-${columnCount}`"
            :key="columnCount"
            :src="getColumnImage(columnCount)"
            @click="changeColumns(columnCount)"
            @shortkey="changeColumns(columnCount)"
            v-shortkey.once="[`f${columnCount}`]"
            :class="{
              selected: columns === columnCount,
              'dark-png': store.state.theme === Theme.DARK,
            }"
          />
        </div>
        <div>
          <v-icon
            :id="SHORTCUT_FIND.elementId"
            class="cursor-pointer"
            color="primary"
            icon="mdi-magnify"
            size="x-large"
            @click="openFind()"
          ></v-icon>
          <v-icon
            v-shortkey.once="SHORTCUT_CLEAR_MESSAGES.key"
            @shortkey="clearMessages"
            :id="SHORTCUT_CLEAR_MESSAGES.elementId"
            class="cursor-pointer"
            color="primary"
            icon="mdi-broom"
            size="x-large"
            @click="clearMessages()"
          ></v-icon>
          <v-icon
            v-shortkey.once="SHORTCUT_SETTINGS.key"
            @shortkey="openSettingsModal"
            :id="SHORTCUT_SETTINGS.elementId"
            class="cursor-pointer"
            color="primary"
            icon="mdi-cog"
            size="x-large"
            @click="openSettingsModal()"
          ></v-icon>
          <v-icon
            v-shortkey.once="SHORTCUT_SHORTCUT_GUIDE.key"
            @shortkey="toggleShortcutGuide"
            :id="SHORTCUT_SHORTCUT_GUIDE.elementId"
            class="cursor-pointer"
            color="primary"
            icon="mdi-help"
            size="x-large"
            @click="toggleShortcutGuide()"
          ></v-icon>
        </div>
      </div>
      <FindModal ref="findRef"></FindModal>
    </header>

    <main class="content">
      <div id="content">
        <ChatMessages :columns="columns"></ChatMessages>
      </div>
    </main>

    <FooterBar></FooterBar>
    <SettingsModal v-model:open="isSettingsOpen" />
    <ConfirmModal ref="confirmModal" />
    <UpdateNotification></UpdateNotification>
    <ShortcutGuide
      ref="shortcutGuideRef"
      v-model:open="isShortcutGuideOpen"
    ></ShortcutGuide>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from "vue";

import { useTheme } from "vuetify";
import { useStore } from "vuex";
import { v4 as uuidv4 } from "uuid";
import { applyTheme, resolveTheme, Theme } from "./theme";
import {
  SHORTCUT_FIND,
  SHORTCUT_SETTINGS,
  SHORTCUT_SHORTCUT_GUIDE,
  SHORTCUT_CLEAR_MESSAGES,
} from "./components/ShortcutGuide/shortcut.const";

import i18n from "./i18n";

// Components
import ChatMessages from "@/components/Messages/ChatMessages.vue";
import SettingsModal from "@/components/SettingsModal.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import FooterBar from "@/components/Footer/FooterBar.vue";
import UpdateNotification from "@/components/Notification/UpdateNotificationModal.vue";
import FindModal from "@/components/FindModal.vue";
import ShortcutGuide from "@/components/ShortcutGuide/ShortcutGuide.vue";

// Styles
import "@mdi/font/css/materialdesignicons.css";

const { ipcRenderer } = window.require("electron");

const store = useStore();
const vuetifyTheme = useTheme();
const onUpdatedSystemTheme = async () => {
  const resolvedTheme = await resolveTheme(store.state.mode, ipcRenderer);
  store.commit("setTheme", resolvedTheme);
  applyTheme(resolvedTheme, vuetifyTheme);
};

ipcRenderer.on("on-updated-system-theme", onUpdatedSystemTheme);

const confirmModal = ref(null);
const findRef = ref(null);
const shortcutGuideRef = ref(null);
const isShortcutGuideOpen = ref(false);
const isSettingsOpen = ref(false);

const columns = computed(() => store.state.columns);

const changeColumns = (columns) => store.commit("changeColumns", columns);
const setUuid = (uuid) => store.commit("setUuid", uuid);

async function openSettingsModal() {
  if (isSettingsOpen.value) {
    // click too fast
    isSettingsOpen.value = false;
    await nextTick();
  }
  isSettingsOpen.value = true;
}

function openFind() {
  findRef.value.showFindTextField();
}

function toggleShortcutGuide() {
  shortcutGuideRef.value.toggleShortcutGuide();
}

async function clearMessages() {
  const result = await confirmModal.value.showModal(
    i18n.global.t("header.clearMessages"),
  );
  if (result) {
    store.commit("clearMessages");
  }
}

onMounted(() => {
  !store.state.uuid && setUuid(uuidv4());
  window._paq.push(["setUserId", store.state.uuid]);
  window._paq.push(["trackPageView"]);

  const ver = require("../package.json").version;
  document.title = `ChatALL.ai - v${ver}`;
});

function getColumnImage(columnCount) {
  return require(`@/assets/column-${columnCount}.svg`);
}
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: "Arial", sans-serif;
}

#app {
    display: flex;
    flex-direction: column;
    height: 100vh;
}

header {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    background-color: rgb(var(--v-theme-header));
    box-shadow: 0 4px 4px rgba(0, 0, 0, 0.25);
    padding: 16px;
    z-index: 999;
}

.header-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.logo {
  height: 40px;
}

.column-icons img {
    opacity: 0.5;
    cursor: pointer;
    width: 24px;
    height: 24px;
    margin: 4px;
}

img.selected {
  opacity: 1;
}

.content {
    flex: 1;
    background-color: rgb(var(--v-theme-background));
    padding: 16px;
}

.cursor-pointer {
    cursor: pointer;
}

.dark-png {
  filter: grayscale(1) brightness(5);
}
</style>
