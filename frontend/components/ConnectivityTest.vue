<template>
  <!-- Connectivity -->
  <div class="availability-test-section mb-4">
    <div class="jn-title2 d-flex justify-content-between align-items-center">
      <h2 id="Connectivity" :class="{ 'mobile-h2': isMobile }">🚦 {{ t('connectivity.Title') }}</h2>
      <button 
        @click="toggleVisibility" 
        class="btn btn-sm"
        :class="[isDarkMode ? 'btn-outline-light' : 'btn-outline-dark']"
        v-tooltip="{ title: isVisible ? t('connectivity.Hide') : t('connectivity.Show'), placement: 'top' }"
      >
        <i :class="isVisible ? 'bi bi-chevron-up' : 'bi bi-chevron-down'"></i>
      </button>
    </div>
    <Transition name="slide-fade">
      <div v-if="isVisible">
        <div class="d-flex justify-content-between align-items-center mb-3">
          <div class="text-secondary">
            <p>{{ t('connectivity.Note') }}</p>
          </div>
          <button @click="checkAllConnectivity(false, true, true)"
            :class="['btn', isDarkMode ? 'btn-dark dark-mode-refresh' : 'btn-light']" aria-label="Refresh Connectivity Test"
            v-tooltip="t('Tooltips.RefreshConnectivityTests')">
            <i class="bi" :class="[isStarted ? 'bi-arrow-clockwise' : 'bi-caret-right-fill']"></i>
          </button>
        </div>
        <div class="row">
          <div v-for="test in connectivityTests" :key="test.id" class="col-6 col-md-3 mb-4">
            <div class="card jn-card keyboard-shortcut-card"
              :class="{ 'dark-mode dark-mode-border': isDarkMode, 'jn-hover-card': !isMobile }">
              <div class="card-body">
                <p class="jn-con-title card-title" @click.prevent="checkConnectivityHandler(test, onTestComplete, true)"
                  :title="t('connectivity.RefreshThisTest')"><i class="bi" :class="'bi-' + test.icon"></i> {{ test.name }}
                </p>
                <p class="card-text" :class="{
                    'text-info': test.status === t('connectivity.StatusWait'),
                    'text-success': test.status.includes(t('connectivity.StatusAvailable')) && test.time < 200,
                    'jn-text-warning': test.status.includes(t('connectivity.StatusAvailable')) && test.time >= 200,
                    'text-danger': test.status === t('connectivity.StatusUnavailable') || test.status === t('connectivity.StatusTimeout')
                  }" :title="t('connectivity.minTestTime') + test.mintime + ' ms'">
                  <i v-if="test.status === t('connectivity.StatusUnavailable') || test.status === t('connectivity.StatusTimeout')"
                    class="bi bi-emoji-frown"></i>
                  <i v-else-if="test.status === t('connectivity.StatusAvailable') && test.time < 200"
                    class="bi bi-emoji-smile"></i>
                  <i v-else-if="test.status === t('connectivity.StatusAvailable') && test.time >= 200"
                    class="bi bi-emoji-expressionless"></i>
                  <i v-else-if="test.time === 0" class="bi bi-hourglass-split"></i>
                  {{ test.status }}
                  <span v-if="test.time !== 0">
                    : {{ test.time }}
                    <span> ms</span>
                  </span>
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, reactive, watch, onUnmounted } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';
import { trackEvent } from '@/utils/use-analytics';

const { t } = useI18n();

const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);
const userPreferences = computed(() => store.userPreferences);

const alertToShow = ref(false);
const alertStyle = ref("");
const alertTitle = ref("");
const alertMessage = ref("");
const autoRefresh = computed(() => userPreferences.value.connectivityAutoRefresh);
const autoShowAltert = computed(() => userPreferences.value.popupConnectivityNotifications);
const isStarted = ref(false);
const counter = ref(0);
const maxCounts = ref(5);
const manualRun = ref(false);
const refreshIntervalId = ref(null);
const isVisible = ref(true);

const connectivityTests = reactive([
  {
    id: "taobao",
    name: "Taobao",
    icon: "shop",
    url: "https://www.taobao.com/favicon.ico?",
    status: t('connectivity.StatusWait'),
    time: 0,
    mintime: 0,
  },
  {
    id: "baidu",
    name: "Baidu",
    icon: "browser-safari",
    url: "https://www.baidu.com/favicon.ico?",
    status: t('connectivity.StatusWait'),
    time: 0,
    mintime: 0,
  },
  {
    id: "wechat",
    name: "WeChat",
    icon: "wechat",
    url: "https://res.wx.qq.com/a/wx_fed/assets/res/NTI4MWU5.ico?",
    status: t('connectivity.StatusWait'),
    time: 0,
    mintime: 0,
  },
  {
    id: "google",
    name: "Google",
    icon: "google",
    url: "https://www.google.com/favicon.ico?",
    status: t('connectivity.StatusWait'),
    time: 0,
    mintime: 0,
  },
  {
    id: "cloudflare",
    name: "Cloudflare",
    icon: "cloud-fill",
    url: "https://www.cloudflare.com/favicon.ico?",
    status: t('connectivity.StatusWait'),
    time: 0,
    mintime: 0,
  },
  {
    id: "youtube",
    name: "YouTube",
    icon: "youtube",
    url: "https://www.youtube.com/favicon.ico?",
    status: t('connectivity.StatusWait'),
    time: 0,
    mintime: 0,
  },
  {
    id: "github",
    name: "GitHub",
    icon: "github",
    url: "https://github.com/favicon.ico?",
    status: t('connectivity.StatusWait'),
    time: 0,
    mintime: 0,
  },
  {
    id: "chatgpt",
    name: "ChatGPT",
    icon: "chat-quote-fill",
    url: "https://chatgpt.com/favicon.ico?",
    status: t('connectivity.StatusWait'),
    time: 0,
    mintime: 0,
  },
]);

// 添加切换可见性的方法
const toggleVisibility = () => {
  isVisible.value = !isVisible.value;
};

// 检查网络连通性
const checkConnectivityHandler = (test, onTestComplete = () => { }, isManualRun) => {
  const beginTime = +new Date();
  manualRun.value = isManualRun;
  let img = new Image();
  let timeout = setTimeout(() => {
    test.status = t('connectivity.StatusUnavailable');
    onTestComplete(false);
  }, 2000); // 减少超时时间到2秒

  img.onload = () => {
    clearTimeout(timeout);
    test.status = t('connectivity.StatusAvailable');
    let testTime = new Date() - beginTime;

    if (test.mintime === 0) {
      test.mintime = testTime;
    } else {
      test.mintime = Math.min(test.mintime, testTime);
    }

    if (autoRefresh.value && !isManualRun) {
      test.time = test.mintime;
    } else {
      test.time = testTime;
    }
    onTestComplete(true);
  };

  img.onerror = () => {
    clearTimeout(timeout);
    test.time = 0;
    test.status = t('connectivity.StatusUnavailable');
    onTestComplete(false);
  };

  img.src = `${test.url}${Date.now()}`;
};

// 检查所有网络连通性
const checkAllConnectivity = (isAlertToShow, isRefresh, isManualRun) => {
  alertToShow.value = isAlertToShow;
  return new Promise((resolve) => {
    if (isRefresh) {
      connectivityTests.forEach((test) => {
        test.status = t('connectivity.StatusWait');
        test.time = 0;
      });
      trackEvent('Section', 'RefreshClick', 'Connectivity');
    }

    let totalTests = connectivityTests.length;
    let successCount = 0;
    let testPromises = [];

    const onTestComplete = (isSuccess) => {
      if (isSuccess) {
        successCount++;
      }
    };

    connectivityTests.forEach((test, index) => {
      let testPromise = new Promise((testResolve, testReject) => {
        setTimeout(() => {
          checkConnectivityHandler(test, (isSuccess) => {
            if (isSuccess) {
              onTestComplete(true);
              testResolve();
            } else {
              onTestComplete(false);
              testReject();
            }
          }, isManualRun);
        }, 50 * index);
      });
      testPromises.push(testPromise);
    });

    // 无论如何都会 Resolve
    Promise.allSettled(testPromises).then(() => {
      if (successCount === totalTests) {
        updateConnectivityAlert("success");
      } else {
        updateConnectivityAlert("error");
      }
      resolve();
    });

    isStarted.value = true;
  });
};

// 更新通知气泡
const updateConnectivityAlert = (type) => {
  if (type === "success") {
    alertStyle.value = "text-success";
    alertMessage.value = t('alert.Congrats_Message');
    alertTitle.value = t('alert.Congrats');
  } else {
    alertStyle.value = "text-danger";
    alertMessage.value = t('alert.OhNo_Message');
    alertTitle.value = t('alert.OhNo');
  }
  sendAlert();
};

const sendAlert = () => {
  if ((alertToShow.value || !isStarted.value) && autoShowAltert.value) {
    store.setAlert(alertToShow.value, alertStyle.value, alertMessage.value, alertTitle.value);
  }
};

// 主控制函数
const handelCheckStart = async (fromApp = false) => {
  // 重置计数器
  counter.value = 0;

  if (fromApp) {
    await checkAllConnectivity(false, true, false);
  } else {
    await checkAllConnectivity(true, false, false);
  }

  // 传递完成状态到 store
  store.setLoadingStatus('connectivity', true);

  // 清除之前的定时器
  if (refreshIntervalId.value) {
    clearInterval(refreshIntervalId.value);
    refreshIntervalId.value = null;
  }

  if (autoRefresh.value) {
    refreshIntervalId.value = setInterval(async () => {
      if (counter.value < maxCounts.value && !manualRun.value) {
        await checkAllConnectivity(false, false, false);
        counter.value++;
      } else {
        clearInterval(refreshIntervalId.value);
        refreshIntervalId.value = null;
      }
    }, 3000);
  }
};

onMounted(() => {
  store.setMountingStatus('connectivity', true);
});

watch(() => store.allHasLoaded, (newValue) => {
  if (newValue === true) {
    sendAlert();
  }
});

// 组件卸载时清理定时器
onUnmounted(() => {
  if (refreshIntervalId.value) {
    clearInterval(refreshIntervalId.value);
  }
});

defineExpose({
  checkAllConnectivity,
  handelCheckStart,
});
</script>

<style scoped>
.availability-test-section {
  transition: all 0.3s ease;
}

.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.3s ease;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

.jn-card {
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.jn-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.dark-mode {
  background-color: #2c2c2c;
  color: #ffffff;
}

.dark-mode-border {
  border-color: #404040;
}

.jn-con-title {
  font-size: 1.1rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.jn-con-title i {
  font-size: 1.2rem;
}

.card-text {
  font-size: 0.9rem;
  margin-bottom: 0;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.jn-text-warning {
  color: #ffc107;
}

.dark-mode-refresh {
  background-color: #404040;
  border-color: #505050;
}

.dark-mode-refresh:hover {
  background-color: #505050;
}

@media (max-width: 768px) {
  .mobile-h2 {
    font-size: 1.5rem;
  }
  
  .jn-card {
    margin-bottom: 1rem;
  }
  
  .card-text {
    font-size: 0.8rem;
  }
}

@media (max-width: 576px) {
  .col-6 {
    width: 100%;
  }
}
</style>


