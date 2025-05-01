<template>
  <div>
    <!-- WebRTC Test -->
    <div class="webrtc-test-section mb-4">
      <div class="jn-title2 d-flex justify-content-between align-items-center">
        <h2 id="WebRTC" :class="{ 'mobile-h2': isMobile }">🚥 {{ t('webrtc.Title') }}</h2>
        <button 
          @click="toggleVisibility" 
          class="btn btn-sm"
          :class="[isDarkMode ? 'btn-outline-light' : 'btn-outline-dark']"
          v-tooltip="{ title: isVisible ? t('webrtc.Hide') : t('webrtc.Show'), placement: 'top' }"
        >
          <i :class="isVisible ? 'bi bi-chevron-up' : 'bi bi-chevron-down'"></i>
        </button>
      </div>
      <Transition name="slide-fade">
        <div v-if="isVisible">
          <div class="d-flex justify-content-between align-items-center mb-3">
            <div class="text-secondary">
              <p>{{ t('webrtc.Note') }}</p>
            </div>
            <button @click="checkAllWebRTC(true)" :class="['btn', isDarkMode ? 'btn-dark dark-mode-refresh' : 'btn-light']"
              aria-label="Refresh WebRTC Test" v-tooltip="t('Tooltips.RefreshWebRTC')">
              <i class="bi" :class="[isStarted ? 'bi-arrow-clockwise' : 'bi-caret-right-fill']"></i>
            </button>
          </div>
          <div v-if="isIPLeaked" class="alert alert-warning mb-3" role="alert">
            <i class="bi bi-exclamation-triangle-fill me-2"></i>
            {{ t('webrtc.IPLeakWarning') }}
          </div>
          <div class="row">
            <div v-for="(stun, index) in stunServers" :key="stun.id" class="col-lg-3 col-md-6 col-12 mb-4"
              :class="{ 'd-none': index >= webrtcServerCount }">
              <div class="card jn-card keyboard-shortcut-card"
                :class="{ 'dark-mode dark-mode-border': isDarkMode, 'jn-hover-card': !isMobile }">
                <div class="card-body">
                  <p class="card-title jn-con-title"><i class="bi bi-sign-merge-left-fill"></i> {{ stun.name }}</p>
                  <p class="card-text text-secondary" style="font-size: 10pt;"><i class="bi bi-hdd-network-fill"></i> {{
                    stun.url }}</p>
                  <p class="card-text" :class="{
                    'text-info': stun.ip === t('webrtc.StatusWait'),
                    'text-success': stun.ip.includes('.') || stun.ip.includes(':'),
                    'text-danger': stun.ip === t('webrtc.StatusError')
                  }">
                    <i class="bi"
                      :class="[stun.ip === t('webrtc.StatusWait') ? 'bi-hourglass-split' : 'bi-pc-display-horizontal']">&nbsp;</i>
                    <span :class="{ 'jn-ip-font': stun.ip.length > 32 }"> {{ stun.ip }}
                    </span>
                  </p>
                  <div v-if="stun.natType" class="alert d-flex flex-column" :class="{
                    'alert-info': stun.natType === t('webrtc.StatusWait'),
                    'alert-success': stun.natType !== t('webrtc.StatusWait'),
                  }" :data-bs-theme="isDarkMode ? 'dark' : ''">
                    <span>
                      <i class="bi"
                        :class="[stun.natType === t('webrtc.StatusWait') ? 'bi-hourglass-split' : ' bi-controller']"></i> NAT:
                      {{ stun.natType }}
                    </span>

                    <div class="mt-2">
                      <div class="d-flex align-items-center mb-1 location-clickable" @click.stop="showLocation(stun)" v-tooltip="t('Tooltips.ShowOnMap')">
                        <i class="bi bi-geo-alt-fill me-2"></i>
                        <span class="text-break">{{ stun.city }}{{ stun.region ? ', ' + stun.region : '' }}</span>
                      </div>
                      <div class="d-flex align-items-center mb-1">
                        <i class="bi bi-flag-fill me-2"></i>
                        <span class="text-break">{{ stun.country_name }}</span>
                        <span v-show="stun.country_code" :class="['jn-fl', 'fi', 'fi-' + stun.country_code.toLowerCase(), 'ms-2']"></span>
                      </div>
                      <div class="d-flex align-items-center">
                        <i class="bi bi-building me-2"></i>
                        <span class="text-break">{{ stun.org }}</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </div>
    <MapModal ref="mapModal" :location="currentLocation" />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, reactive, watch, nextTick } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';
import { trackEvent } from '@/utils/use-analytics';
import { transformDataFromIPapi } from '@/utils/transform-ip-data.js';
import getCountryName from '@/utils/country-name.js';
import MapModal from './MapModal.vue';

const { t } = useI18n();

const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);
const lang = computed(() => store.lang);
const webrtcServerCount = computed(() => store.userPreferences.webrtcServerCount || 4);
const mainIP = computed(() => store.allIPs[0]); // 获取主IP地址

const isStarted = ref(false);
const IPArray = ref([]);
const isIPLeaked = ref(false);
const stunServers = reactive([
  // 原有的服务器
  {
    id: "google",
    name: "Google",
    url: "stun.l.google.com:19302",
    ip: t('webrtc.StatusWait'),
    natType: t('webrtc.StatusWait'),
    city: t('webrtc.StatusWait'),
    region: t('webrtc.StatusWait'),
    country: t('webrtc.StatusWait'),
    country_name: t('webrtc.StatusWait'),
    country_code: '',
    org: t('webrtc.StatusWait'),
  },
  {
    id: "blackberry",
    name: "BlackBerry",
    url: "stun.voip.blackberry.com:3478",
    ip: t('webrtc.StatusWait'),
    natType: t('webrtc.StatusWait'),
    city: t('webrtc.StatusWait'),
    region: t('webrtc.StatusWait'),
    country: t('webrtc.StatusWait'),
    country_name: t('webrtc.StatusWait'),
    country_code: '',
    org: t('webrtc.StatusWait'),
  },
  {
    id: "twilio",
    name: "Twilio",
    url: "global.stun.twilio.com",
    ip: t('webrtc.StatusWait'),
    natType: t('webrtc.StatusWait'),
    city: t('webrtc.StatusWait'),
    region: t('webrtc.StatusWait'),
    country: t('webrtc.StatusWait'),
    country_name: t('webrtc.StatusWait'),
    country_code: '',
    org: t('webrtc.StatusWait'),
  },
  {
    id: "cloudflare",
    name: "Cloudflare",
    url: "stun.cloudflare.com",
    ip: t('webrtc.StatusWait'),
    natType: t('webrtc.StatusWait'),
    city: t('webrtc.StatusWait'),
    region: t('webrtc.StatusWait'),
    country: t('webrtc.StatusWait'),
    country_name: t('webrtc.StatusWait'),
    country_code: '',
    org: t('webrtc.StatusWait'),
  },
  {
    id: "microsoft",
    name: "Microsoft",
    url: "stun.sipgate.net:3478",
    ip: t('webrtc.StatusWait'),
    natType: t('webrtc.StatusWait'),
    city: t('webrtc.StatusWait'),
    region: t('webrtc.StatusWait'),
    country: t('webrtc.StatusWait'),
    country_name: t('webrtc.StatusWait'),
    country_code: '',
    org: t('webrtc.StatusWait'),
  },
  {
    id: "amazon",
    name: "Amazon",
    url: "stun.kinesisvideo.us-east-1.amazonaws.com:443",
    ip: t('webrtc.StatusWait'),
    natType: t('webrtc.StatusWait'),
    city: t('webrtc.StatusWait'),
    region: t('webrtc.StatusWait'),
    country: t('webrtc.StatusWait'),
    country_name: t('webrtc.StatusWait'),
    country_code: '',
    org: t('webrtc.StatusWait'),
  },
  {
    id: "xiaomi",
    name: "Xiaomi",
    url: "stun.miwifi.com:3478",
    ip: t('webrtc.StatusWait'),
    natType: t('webrtc.StatusWait'),
    city: t('webrtc.StatusWait'),
    region: t('webrtc.StatusWait'),
    country: t('webrtc.StatusWait'),
    country_name: t('webrtc.StatusWait'),
    country_code: '',
    org: t('webrtc.StatusWait'),
  },
  {
    id: "nextCloud",
    name: "NextCloud",
    url: "stun.nextcloud.com:3478",
    ip: t('webrtc.StatusWait'),
    natType: t('webrtc.StatusWait'),
    city: t('webrtc.StatusWait'),
    region: t('webrtc.StatusWait'),
    country: t('webrtc.StatusWait'),
    country_name: t('webrtc.StatusWait'),
    country_code: '',
    org: t('webrtc.StatusWait'),
  }
]);

const mapModal = ref(null);
const currentLocation = ref({
  latitude: 0,
  longitude: 0,
  address: '',
  country: '',
  organization: ''
});

// 添加显示/隐藏状态
const isVisible = ref(true);

// 添加切换可见性的方法
const toggleVisibility = () => {
  isVisible.value = !isVisible.value;
};

// 测试 STUN 服务器
const checkSTUNServer = async (stun) => {
  return new Promise((resolve, reject) => {
    try {
      const servers = { iceServers: [{ urls: 'stun:' + stun.url }] };
      const pc = new RTCPeerConnection(servers);
      let candidateReceived = false;

      // 分别获取 STUN 服务器的 IP 地址和 NAT 类型
      pc.onicecandidate = async (event) => {
        if (event.candidate) {
          candidateReceived = true;
          const candidate = event.candidate.candidate;
          const ipMatch = /([0-9a-f]{1,4}(:[0-9a-f]{1,4}){7}|[0-9a-f]{0,4}(:[0-9a-f]{1,4}){0,6}::[0-9a-f]{0,4}|::[0-9a-f]{1,4}(:[0-9a-f]{1,4}){0,6}|[0-9]{1,3}(\.[0-9]{1,3}){3})/i.exec(candidate);
          if (ipMatch) {
            stun.ip = ipMatch[0];
            try {
              let countryInfo = await fetchCountryCode(stun.ip);
              stun.country_code = countryInfo.country_code;
              stun.country_name = countryInfo.country_name;
              stun.city = countryInfo.city;
              stun.region = countryInfo.region;
              stun.org = countryInfo.org;
              stun.latitude = countryInfo.latitude;
              stun.longitude = countryInfo.longitude;
            } catch (error) {
              console.error("Error fetching country code:", error);
              reject(error);
              pc.close();
              return;
            }
            IPArray.value = [...IPArray.value, stun.ip];
            stun.natType = determineNATType(candidate);
            pc.close();
            resolve();
          }
        }
      };

      pc.createDataChannel("");
      pc.createOffer().then((offer) => pc.setLocalDescription(offer));

      // 设置一个超时计时器来拒绝 Promise
      setTimeout(() => {
        if (!candidateReceived) {
          pc.close();
          reject(new Error("Stun Server Test Timeout"));
        }
      }, 5000);
    } catch (error) {
      console.error("STUN Server Test Error:", error);
      stun.ip = t('webrtc.StatusError');
      reject(error);
    }
  });
};


// 分析ICE候选信息，推断NAT类型
const determineNATType = (candidate) => {
  const parts = candidate.split(' ');
  const type = parts[7];

  if (type === 'host') {
    return t('webrtc.NATType.host');
  } else if (type === 'srflx') {
    return t('webrtc.NATType.srflx');
  } else if (type === 'prflx') {
    return t('webrtc.NATType.prflx');
  } else if (type === 'relay') {
    return t('webrtc.NATType.relay');
  } else {
    return t('webrtc.NATType.unknown');
  }
};

// 通过 Maxmind 获取 IP 地区归属
const fetchCountryCode = async (ip) => {
  let setLang = lang.value;
  if (setLang === 'zh') {
    setLang = 'zh-CN';
  }

  try {
    const response = await fetch(`/l-api/geoIp?ip=${ip}&lang=${setLang}`);
    const data = await response.json();
    
    if (data) {
      return {
        country_code: data.country_code?.toLowerCase() || '',
        country_name: data.country_name || 'N/A',
        city: data.city || 'N/A',
        region: data.region || 'N/A',
        org: data.org || 'N/A',
        latitude: data.latitude || 0,
        longitude: data.longitude || 0
      };
    }
  } catch (error) {
    console.error("Error fetching IP country code", error);
    return {
      country_code: '',
      country_name: t('webrtc.StatusError'),
      city: t('webrtc.StatusError'),
      region: t('webrtc.StatusError'),
      org: t('webrtc.StatusError'),
      latitude: 0,
      longitude: 0
    };
  }
}


// 测试所有 STUN 服务器
const checkAllWebRTC = async (isRefresh) => {
  if (isRefresh) {
    trackEvent('Section', 'RefreshClick', 'WebRTC');
  }
  isStarted.value = true;
  const visibleServers = stunServers.slice(0, webrtcServerCount.value);
  const promises = visibleServers.map((server) => {
    server.ip = t('webrtc.StatusWait');
    server.natType = t('webrtc.StatusWait');
    server.country = t('webrtc.StatusWait');
    server.country_code = '';
    return checkSTUNServer(server);
  });

  const allSettledPromise = Promise.allSettled(promises);
  const timeoutPromise = new Promise((resolve) => setTimeout(resolve, 6000));

  return Promise.race([allSettledPromise, timeoutPromise]).then(() => {
    store.setLoadingStatus('webrtc', true);
  });
};

// 检查IP是否泄露
const checkIPLeak = () => {
  if (!mainIP.value) return;
  
  const stunIPs = stunServers
    .slice(0, webrtcServerCount.value)
    .map(server => server.ip)
    .filter(ip => ip !== t('webrtc.StatusWait') && ip !== t('webrtc.StatusError'));
  
  isIPLeaked.value = stunIPs.some(ip => ip !== mainIP.value);
};

const showLocation = (stun) => {
  if (!stun || !mapModal.value) return;
  
  // 检查是否有有效的坐标信息
  if (!stun.latitude || !stun.longitude || 
      stun.latitude === 0 || stun.longitude === 0 ||
      stun.city === t('webrtc.StatusError') || 
      stun.country_name === t('webrtc.StatusError')) {
    return;
  }
  
  // 使用后端返回的坐标数据
  currentLocation.value = {
    latitude: stun.latitude,
    longitude: stun.longitude,
    address: `${stun.city}${stun.region ? ', ' + stun.region : ''}`,
    country: stun.country_name,
    organization: stun.org
  };
  
  // 确保模态框已经挂载
  nextTick(() => {
    mapModal.value.show();
  });
};

onMounted(() => {
  store.setMountingStatus('webrtc', true);
});

watch(IPArray, () => {
  store.updateAllIPs(IPArray.value);
  checkIPLeak();
}, { deep: true });

defineExpose({
  checkAllWebRTC,
  stunServers
});

</script>

<style scoped>
.location-clickable {
  cursor: pointer;
  transition: all 0.2s ease;
  border-radius: 4px;
}

.location-clickable:hover {
  color: var(--bs-primary);
  background-color: rgba(var(--bs-primary-rgb), 0.1);
}

.location-clickable:hover i {
  transform: scale(1.1);
}

/* 添加按钮样式 */
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

/* 调整标题样式 */
.jn-title2 {
  margin-bottom: 1rem;
}

/* 添加动画样式 */
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
</style>
