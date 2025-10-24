<script setup>
import { darkTheme, useOsTheme } from 'naive-ui'
import { list as langList, setI18nLanguage, loadLocaleMessages, autoLang } from './config/lang.js'
import { useAppStore } from './stores/app'
import LoadingCard from '@/components/Loading.vue'
import InfoCard from '@/components/Information.vue'
import SpeedtestCard from '@/components/Speedtest.vue'
import UtilitiesCard from '@/components/Utilities.vue'
import TrafficCard from '@/components/TrafficDisplay.vue'
import { computed } from 'vue'
import { MoonOutline, SunnyOutline } from '@vicons/ionicons5'

const lang = computed(() => {
  return currentLang.value.uiLang()
})

const dateLang = computed(() => {
  return currentLang.value.dateLang()
})

const currentLangCode = ref('en-US')
const currentLang = computed(() => {
  for (var i in langList) {
    const _lang = langList[i]
    if (_lang.value == currentLangCode.value) {
      return _lang
    }
  }
  return null
})

const langDropdown = computed(() => {
  return langList.map((item) => {
    return {
      label: item.label,
      value: item.value
    }
  })
})

const handleLangChange = async () => {
  await loadLocaleMessages(currentLangCode.value)
  setI18nLanguage(currentLangCode.value)
}

const osThemeRef = useOsTheme()
const userTheme = ref(localStorage.getItem('theme') || 'auto')
const theme = computed(() => {
  if (userTheme.value === 'dark') return darkTheme
  if (userTheme.value === 'light') return null
  return osThemeRef.value === 'dark' ? darkTheme : null // auto 模式：跟随系统
})

const toggleTheme = () => {
  userTheme.value =
    userTheme.value === 'auto'
      ? osThemeRef.value === 'dark' ? 'light' : 'dark'
      : userTheme.value === 'dark'
      ? 'light'
      : 'dark'
  localStorage.setItem('theme', userTheme.value)
}
const appStore = useAppStore()

onMounted(async () => {
  currentLangCode.value = await autoLang()
})
</script>

<template>
  <n-config-provider :locale="lang" :date-locale="dateLang" :theme="theme">
    <n-global-style />
    <n-message-provider>
      <n-space vertical>
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <h2 style="margin: 0;">
            Wingmark Looking Glass
            <template v-if="appStore.config?.server_id">
              - {{ appStore.config.server_id }}
            </template>
          </h2>
          <n-button circle @click="toggleTheme" type="primary" quaternary>
            <n-icon size="20">
              <component :is="theme === darkTheme ? SunnyOutline : MoonOutline" />
            </n-icon>
          </n-button>
        </div>
        <LoadingCard v-if="appStore.connecting" />
        <template v-else>
          <InfoCard />
          <UtilitiesCard />
          <SpeedtestCard />
          <TrafficCard v-if="appStore.config.feature_iface_traffic" />
        </template>
        <n-space justify="space-between">
          <div>
            <div style="margin-top: 10px">
              Copyright © 2025 Wingmark Matrix (Wuhan) Computer Systems Co., Ltd. <br/>
              Powered by
              <n-button
                text
                tag="a"
                target="_blank"
                href="https://github.com/wikihost-opensource/als"
              >
                WIKIHOST Opensource - ALS (Github)
              </n-button>
            </div>
            <div>
              <p>{{ $t('memory_usage') }}: {{ appStore.memoryUsage }}</p>
            </div>
          </div>
          <div>
            <n-select
              v-model:value="currentLangCode"
              :options="langDropdown"
              style="min-width: 150px"
              @update:value="handleLangChange"
            />
          </div>
        </n-space>
      </n-space>
    </n-message-provider>
  </n-config-provider>
</template>
