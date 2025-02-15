<template>
  <v-dialog
    :model-value="props.open"
    fullscreen
    :scrim="false"
    transition="dialog-bottom-transition"
  >
    <v-card>
      <v-toolbar dark color="primary">
        <v-toolbar-title>{{ $t("settings.title") }}</v-toolbar-title>
        <v-spacer></v-spacer>

        <v-btn icon dark @click="closeDialog">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-toolbar>
      <v-row>
        <v-col cols="2">
          <v-tabs v-model="tab" direction="vertical">
            <v-tab value="general">{{ $t("settings.general") }}</v-tab>
            <v-tab value="bots">{{ $t("settings.bots") }}</v-tab>
            <v-tab value="proxy">{{ $t("proxy.name") }}</v-tab>
          </v-tabs>
        </v-col>
        <v-col>
          <v-list lines="two" subheader>
            <div class="section" v-if="tab == 'general'">
              <v-list-subheader>{{ $t("settings.general") }}</v-list-subheader>
              <v-list-item>
                <v-list-item-title>{{
                  $t("settings.language")
                }}</v-list-item-title>
                <v-select
                  :items="languages"
                  item-title="name"
                  item-value="code"
                  hide-details
                  :model-value="lang"
                  @update:model-value="setCurrentLanguage($event)"
                ></v-select>
              </v-list-item>
            </div>
            <div class="section" v-if="tab == 'general'">
              <v-list-item>
                <v-list-item-title>{{
                  $t("settings.theme")
                }}</v-list-item-title>
                <v-select
                  :items="modes"
                  item-title="name"
                  item-value="code"
                  hide-details
                  :model-value="currentMode"
                  @update:model-value="setCurrentMode($event)"
                ></v-select>
              </v-list-item>
            </div>

            <template v-for="(setting, index) in settings" :key="index">
              <v-divider v-if="tab == 'bots'"></v-divider>
              <div class="section" v-if="tab == 'bots'">
                <component :is="setting"></component>
              </div>
            </template>
            <div class="section" v-if="tab == 'proxy'">
              <component :is="proxy"></component>
            </div>
          </v-list>
        </v-col>
      </v-row>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { computed, ref } from "vue";
import { useStore } from "vuex";
import { useI18n } from "vue-i18n";
import { useTheme } from "vuetify";

import ProxySettings from "@/components/ProxySetting.vue";

import ChatGPTBotSettings from "@/components/BotSettings/ChatGPTBotSettings.vue";
import OpenAIAPIBotSettings from "@/components/BotSettings/OpenAIAPIBotSettings.vue";
import AzureOpenAIAPIBotSettings from "./BotSettings/AzureOpenAIAPIBotSettings.vue";
import BingChatBotSettings from "@/components/BotSettings/BingChatBotSettings.vue";
import SparkBotSettings from "./BotSettings/SparkBotSettings.vue";
import BardBotSettings from "@/components/BotSettings/BardBotSettings.vue";
import MOSSBotSettings from "@/components/BotSettings/MOSSBotSettings.vue";
import WenxinQianfanBotSettings from "@/components/BotSettings/WenxinQianfanBotSettings.vue";
import GradioAppBotSettings from "@/components/BotSettings/GradioAppBotSettings.vue";
import LMSYSBotSettings from "@/components/BotSettings/LMSYSBotSettings.vue";
import HuggingChatBotSettings from "@/components/BotSettings/HuggingChatBotSettings.vue";
import QianWenBotSettings from "@/components/BotSettings/QianWenBotSettings.vue";
import PoeBotSettings from "@/components/BotSettings/PoeBotSettings.vue";
import SkyWorkBotSettings from "@/components/BotSettings/SkyWorkBotSettings.vue";

import { resolveTheme, applyTheme, Mode } from "../theme";

const { ipcRenderer } = window.require("electron");
const { t: $t, locale } = useI18n();
const store = useStore();
const vuetifyTheme = useTheme();

const props = defineProps(["open"]);
const emit = defineEmits(["update:open", "done"]);

const tab = ref(null);

const settings = [
  ChatGPTBotSettings,
  OpenAIAPIBotSettings,
  AzureOpenAIAPIBotSettings,
  WenxinQianfanBotSettings,
  GradioAppBotSettings,
  BardBotSettings,
  BingChatBotSettings,
  HuggingChatBotSettings,
  LMSYSBotSettings,
  MOSSBotSettings,
  PoeBotSettings,
  QianWenBotSettings,
  SparkBotSettings,
  SkyWorkBotSettings,
];

const proxy = ProxySettings;

const languages = computed(() => [
  { name: $t("settings.system"), code: "auto" },
  { name: "Deutsch", code: "de" },
  { name: "English", code: "en" },
  { name: "Español", code: "es" },
  { name: "Français", code: "fr" },
  { name: "Italiano", code: "it" },
  { name: "日本語", code: "ja" },
  { name: "한국어", code: "ko" },
  { name: "Русский", code: "ru" },
  { name: "Tiếng Việt", code: "vi" },
  { name: "简体中文", code: "zh" },
]);

const modes = computed(() => [
  { name: $t("settings.system"), code: Mode.SYSTEM },
  { name: $t("settings.light"), code: Mode.LIGHT },
  { name: $t("settings.dark"), code: Mode.DARK },
]);

const lang = computed(() => store.state.lang);
const currentMode = computed(() => store.state.mode);

const setCurrentLanguage = (lang) => {
  locale.value = lang;
  store.commit("setCurrentLanguage", lang);
};
const setCurrentMode = async (mode) => {
  const resolvedTheme = await resolveTheme(mode, ipcRenderer);
  store.commit("setMode", mode);
  store.commit("setTheme", resolvedTheme);
  applyTheme(resolvedTheme, vuetifyTheme);
};
const closeDialog = () => {
  emit("update:open", false);
  emit("done");
};
</script>

<style scoped>
.section {
  margin-top: 16px;
  margin-bottom: 16px;
  padding: 0 16px;
}

:deep() .v-slider-thumb__label {
  color: rgb(var(--v-theme-font));
}
</style>
