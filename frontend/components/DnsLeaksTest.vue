<template>
  <!-- DNS Leaks Test -->
  <div class="dnsleak-test-section mb-4">
    <div class="jn-title2 d-flex justify-content-between align-items-center">
      <h2 id="DNSLeakTest" :class="{ 'mobile-h2': isMobile }">🛑 {{ t('dnsleaktest.Title') }}</h2>
      <button @click="toggleVisibility" class="btn btn-sm"
        :class="[isDarkMode ? 'btn-outline-light' : 'btn-outline-dark']"
        v-tooltip="{ title: isVisible ? t('dnsleaktest.Hide') : t('dnsleaktest.Show'), placement: 'top' }">
        <i :class="isVisible ? 'bi bi-chevron-up' : 'bi bi-chevron-down'"></i>
      </button>
    </div>
    <Transition name="slide-fade">
      <div v-if="isVisible">
        <div class="d-flex justify-content-between align-items-center mb-3">
          <div class="text-secondary">
            <p>{{ t('dnsleaktest.Note') }}</p>
            <p>{{ t('dnsleaktest.Note2') }}</p>
          </div>
          <div class="d-flex align-items-center">
            <div class="form-check form-switch me-3">
              <input class="form-check-input" type="checkbox" id="useEightCards" v-model="useEightCards">
              <label class="form-check-label" for="useEightCards">{{ t('dnsleaktest.UseEightCards') }}</label>
            </div>
            <button @click="checkAllDNSLeakTest(true)"
              :class="['btn', isDarkMode ? 'btn-dark dark-mode-refresh' : 'btn-light']"
              aria-label="Refresh DNS Leak Test" v-tooltip="t('Tooltips.RefreshDNSLeakTest')">
              <i class="bi"
                :class="[isLoading ? 'bi-arrow-repeat spin' : isStarted ? 'bi-arrow-clockwise' : 'bi-caret-right-fill']"></i>
            </button>
          </div>
        </div>
        <div class="row">
          <div v-for="(leak, index) in leakTest" :key="leak.id" class="col-lg-3 col-md-6 col-12 mb-4">
            <div class="card jn-card keyboard-shortcut-card"
              :class="{ 'dark-mode dark-mode-border': isDarkMode, 'jn-hover-card': !isMobile }">
              <div class="card-body">
                <p class="jn-con-title card-title">
                  <i class="bi bi-heart-pulse-fill"></i> {{ leak.name }}
                  <i class="bi" :class="'bi-' + (index + 1) + '-square'"></i>&nbsp;
                  <small v-if="leak.isLoading" class="text-muted">
                    <i class="bi bi-arrow-repeat spin"></i>
                  </small>
                </p>
                <p class="card-text" :class="{
                  'text-info': leak.ip === waitStatus || leak.ip === errorStatus,
                  'text-success': leak.ip.includes('.') || leak.ip.includes(':'),
                  'text-warning': leak.ip === timeoutStatus
                }">
                  <i class="bi" :class="[
                    leak.ip === waitStatus || leak.ip === errorStatus ? 'bi-hourglass-split' :
                      leak.ip === timeoutStatus ? 'bi-exclamation-triangle' : 'bi-box-arrow-right'
                  ]"></i>
                  {{ t('dnsleaktest.Endpoint') }}: {{ leak.ip }}
                </p>

                <div class="alert d-flex flex-column" :class="{
                  'alert-info': leak.country === waitStatus,
                  'alert-success': leak.country !== waitStatus && leak.country !== errorStatus && leak.country !== timeoutStatus,
                  'alert-warning': leak.country === timeoutStatus,
                  'alert-danger': leak.country === errorStatus
                }" :data-bs-theme="isDarkMode ? 'dark' : ''">

                  <span class="jn-org">
                    <i class="bi" :class="[
                      leak.org === waitStatus || leak.org === errorStatus ? 'bi-hourglass-split' :
                        leak.org === timeoutStatus ? 'bi-exclamation-triangle' : 'bi-geo-alt-fill'
                    ]"></i>
                    {{ t('ipInfos.ISP') }}: <span :title="leak.org">{{ leak.org }}</span>
                  </span>

                  <span class="mt-2">
                    <i class="bi" :class="[
                      leak.country === waitStatus || leak.country === errorStatus ? 'bi-hourglass-split' :
                        leak.country === timeoutStatus ? 'bi-exclamation-triangle' : 'bi-geo-alt-fill'
                    ]"></i>
                    {{ t('ipInfos.Country') }}: <span :class="[leak.country !== waitStatus ? 'fw-bold' : '']">{{
                      leak.country
                    }}&nbsp;</span>
                    <span v-show="leak.country_code" :class="'jn-fl fi fi-' + leak.country_code.toLowerCase()"></span>
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div v-if="showRetryButton" class="d-flex justify-content-center mt-3 mb-3">
          <button @click="checkAllDNSLeakTest(true)" class="btn btn-primary">
            {{ t('Common.Retry') }} <i class="bi bi-arrow-clockwise"></i>
          </button>
        </div>
      </div>
    </Transition>
  </div>
</template>


<script setup>
import { ref, computed, onMounted, reactive, watch } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';
import { trackEvent } from '@/utils/use-analytics';
import countryLookup from 'country-code-lookup';
import getCountryName from '@/utils/country-name.js';

const { t } = useI18n();

const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);
const lang = computed(() => store.lang);

// 添加测试卡片数量选项
const useEightCards = ref(false);

// 状态常量
const waitStatus = computed(() => t('dnsleaktest.StatusWait'));
const errorStatus = computed(() => t('dnsleaktest.StatusError'));
const timeoutStatus = computed(() => t('dnsleaktest.StatusTimeout') || 'Timed out');

// 创建默认卡片数据
const createDefaultCard = () => ({
  name: t('dnsleaktest.Name'),
  country_code: '',
  country: waitStatus.value,
  ip: waitStatus.value,
  org: waitStatus.value,
  isLoading: false,
  hasError: false,
  hasTimedOut: false
});

// 响应式数据
const leakTest = computed(() => {
  const baseCards = [
    { ...createDefaultCard(), id: "ipapi1" },
    { ...createDefaultCard(), id: "ipapi2" },
    { ...createDefaultCard(), id: "sfshark1" },
    { ...createDefaultCard(), id: "sfshark2" },
  ];

  if (useEightCards.value) {
    return [
      ...baseCards,
      { ...createDefaultCard(), id: "ipapi3" },
      { ...createDefaultCard(), id: "ipapi4" },
      { ...createDefaultCard(), id: "sfshark3" },
      { ...createDefaultCard(), id: "sfshark4" },
    ];
  }
  return baseCards;
});

const isStarted = ref(false);
const isLoading = ref(false);
const isVisible = ref(true);
const showRetryButton = ref(false);

// 监听语言变化，更新卡片中的等待状态文本
watch(lang, () => {
  leakTest.value.forEach(card => {
    if (card.country === waitStatus.value || card.ip === waitStatus.value || card.org === waitStatus.value) {
      if (card.country === waitStatus.value) card.country = waitStatus.value;
      if (card.ip === waitStatus.value) card.ip = waitStatus.value;
      if (card.org === waitStatus.value) card.org = waitStatus.value;
    }
  });
}, { deep: true });

// 切换可见性
const toggleVisibility = () => {
  isVisible.value = !isVisible.value;
};

// 生成唯一的随机字符串
const generateUUID = () => {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
    const r = Math.random() * 16 | 0;
    const v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
};

// 生成 32 位随机字符串
const generate32DigitString = () => {
  const uuid = generateUUID();
  const timestamp = Date.now().toString();
  return (timestamp + uuid.replace(/-/g, '')).substring(0, 32);
};

// 生成 14 位随机字符串
const generate14DigitString = () => {
  const uuid = generateUUID();
  return uuid.replace(/-/g, '').substring(0, 14);
};

// DNS 泄露测试 1 - 使用 ip-api.com
const fetchLeakTestIpApiCom = (index) => {
  return new Promise((resolve, reject) => {
    // 设置加载状态
    leakTest.value[index].isLoading = true;

    const urlString = generate32DigitString();
    const url = `https://${urlString}.edns.ip-api.com/json`;

    // 超时控制
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), 5000);

    fetch(url, { signal: controller.signal })
      .then((response) => {
        clearTimeout(timeoutId);
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        return response.json();
      })
      .then((data) => {
        if (data.dns && "geo" in data.dns && "ip" in data.dns) {
          try {
            const geoSplit = data.dns.geo.split(" - ");
            const country = geoSplit[0].trim();
            const countryInfo = countryLookup.byCountry(country);

            if (countryInfo && countryInfo.iso2) {
              leakTest.value[index].country_code = countryInfo.iso2;
              leakTest.value[index].country = getCountryName(leakTest.value[index].country_code, lang.value);
            } else {
              leakTest.value[index].country = country;
              leakTest.value[index].country_code = '';
            }

            leakTest.value[index].org = geoSplit[1] ? geoSplit[1].trim() : '';
            leakTest.value[index].ip = data.dns.ip;
            leakTest.value[index].hasError = false;
            leakTest.value[index].hasTimedOut = false;
          } catch (error) {
            console.error("Error parsing data:", error);
            handleFetchError(index, error);
          }
        } else {
          console.error("Unexpected data structure:", data);
          handleFetchError(index, new Error("Unexpected data structure"));
        }
      })
      .catch((error) => {
        console.error("Error fetching leak test data:", error);
        if (error.name === 'AbortError') {
          handleTimeout(index);
        } else {
          handleFetchError(index, error);
        }
      })
      .finally(() => {
        leakTest.value[index].isLoading = false;
        resolve();
      });
  });
};

// DNS 泄露测试 2 - 使用 surfsharkdns.com
const fetchLeakTestSfSharkCom = (index) => {
  return new Promise((resolve, reject) => {
    // 设置加载状态
    leakTest.value[index].isLoading = true;

    const urlString = generate14DigitString();
    const url = `https://${urlString}.ipv4.surfsharkdns.com`;

    // 超时控制
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), 5000);

    fetch(url, { signal: controller.signal })
      .then((response) => {
        clearTimeout(timeoutId);
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        return response.json();
      })
      .then((data) => {
        // 安全地获取第一个键的数据
        const keys = Object.keys(data);
        if (keys.length > 0) {
          const keyEntry = data[keys[0]];
          if (keyEntry && keyEntry.CountryCode && keyEntry.IP) {
            leakTest.value[index].country_code = keyEntry.CountryCode;
            leakTest.value[index].country = getCountryName(keyEntry.CountryCode, lang.value);
            leakTest.value[index].org = keyEntry.ISP || '';
            leakTest.value[index].ip = keyEntry.IP;
            leakTest.value[index].hasError = false;
            leakTest.value[index].hasTimedOut = false;
          } else {
            handleFetchError(index, new Error("Missing required fields in data"));
          }
        } else {
          handleFetchError(index, new Error("Empty data returned"));
        }
      })
      .catch((error) => {
        console.error("Error fetching leak test data:", error);
        if (error.name === 'AbortError') {
          handleTimeout(index);
        } else {
          handleFetchError(index, error);
        }
      })
      .finally(() => {
        leakTest.value[index].isLoading = false;
        resolve();
      });
  });
};

// 处理超时
const handleTimeout = (index) => {
  leakTest.value[index].ip = timeoutStatus.value;
  leakTest.value[index].country = timeoutStatus.value;
  leakTest.value[index].org = timeoutStatus.value;
  leakTest.value[index].country_code = '';
  leakTest.value[index].hasTimedOut = true;
  showRetryButton.value = true;
};

// 处理错误
const handleFetchError = (index, error) => {
  leakTest.value[index].ip = errorStatus.value;
  leakTest.value[index].country = errorStatus.value;
  leakTest.value[index].org = errorStatus.value;
  leakTest.value[index].country_code = '';
  leakTest.value[index].hasError = true;
  showRetryButton.value = true;
};

// 重置测试状态
const resetTestStatus = () => {
  leakTest.value.forEach((server) => {
    server.country = waitStatus.value;
    server.ip = waitStatus.value;
    server.country_code = '';
    server.org = waitStatus.value;
    server.isLoading = false;
    server.hasError = false;
    server.hasTimedOut = false;
  });
  showRetryButton.value = false;
};

// 检查所有 DNS 泄露测试
const checkAllDNSLeakTest = async (isRefresh) => {
  isStarted.value = true;
  isLoading.value = true;

  if (isRefresh) {
    trackEvent('Section', 'RefreshClick', 'DNSLeakTest');
    resetTestStatus();
  }

  try {
    const tests = [
      fetchLeakTestIpApiCom(0),
      new Promise(resolve => setTimeout(() => fetchLeakTestIpApiCom(1).then(resolve), 300)),
      new Promise(resolve => setTimeout(() => fetchLeakTestSfSharkCom(2).then(resolve), 600)),
      new Promise(resolve => setTimeout(() => fetchLeakTestSfSharkCom(3).then(resolve), 900))
    ];

    if (useEightCards.value) {
      tests.push(
        new Promise(resolve => setTimeout(() => fetchLeakTestIpApiCom(4).then(resolve), 1200)),
        new Promise(resolve => setTimeout(() => fetchLeakTestIpApiCom(5).then(resolve), 1500)),
        new Promise(resolve => setTimeout(() => fetchLeakTestSfSharkCom(6).then(resolve), 1800)),
        new Promise(resolve => setTimeout(() => fetchLeakTestSfSharkCom(7).then(resolve), 2100))
      );
    }

    await Promise.all(tests);

    // 检查是否有任何测试失败
    const hasAnyErrors = leakTest.value.some(test => test.hasError || test.hasTimedOut);
    showRetryButton.value = hasAnyErrors;

  } catch (error) {
    console.error("Error in DNS leak test:", error);
    showRetryButton.value = true;
  } finally {
    isLoading.value = false;
    store.setLoadingStatus('dnsleaktest', true);
  }
};

onMounted(() => {
  store.setMountingStatus('dnsleaktest', true);
});

defineExpose({
  checkAllDNSLeakTest,
  leakTest
});
</script>

<style scoped>
.jn-org {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 按钮样式 */
.btn-outline-dark,
.btn-outline-light {
  padding: 0.25rem 0.5rem;
  border-radius: 50%;
  transition: all 0.3s ease;
}

.btn-outline-dark:hover,
.btn-outline-light:hover {
  transform: scale(1.1);
}

/* 标题样式 */
.jn-title2 {
  margin-bottom: 1rem;
}

/* 动画样式 */
.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateY(-20px);
  opacity: 0;
}

/* 添加加载动画 */
.spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}
</style>