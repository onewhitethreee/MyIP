<template>
    <!-- Whois Resolver -->
    <div class="whois-section my-4">
        <div class="text-secondary">
            <p>{{ t('whois.Note') }}</p>
        </div>
        <div class="row">
            <div class="col-12 mb-3">
                <div class="card jn-card" :class="{ 'dark-mode dark-mode-border': isDarkMode }">
                    <div class="card-body">
                        <div class="col-12 col-md-auto">
                            <label for="queryURLorIP" class="col-form-label">{{ t('whois.Note2') }}</label>
                        </div>

                        <div class="input-group mb-2 mt-2 ">
                            <input type="text" class="form-control" :class="{ 'dark-mode': isDarkMode }"
                                :disabled="whoisCheckStatus === 'running'" :placeholder="t('whois.Placeholder')"
                                v-model="queryURLorIP" @keyup.enter="onSubmit" name="queryURLorIP" id="queryURLorIP"
                                data-1p-ignore>

                            <button class="btn btn-primary" @click="onSubmit"
                                :disabled="whoisCheckStatus === 'running' || !queryURLorIP">
                                <span v-if="whoisCheckStatus === 'idle'">{{
                                    t('whois.Run') }}</span>
                                <span v-if="whoisCheckStatus === 'running'" class="spinner-grow spinner-grow-sm"
                                    aria-hidden="true"></span>
                            </button>

                        </div>

                        <!-- Whois 服务器选择 -->
                        <div class="mt-3">
                            <div class="form-check form-check-inline" v-for="server in availableServers" :key="server">
                                <input class="form-check-input" type="checkbox" :id="'server-' + server"
                                    v-model="selectedServers" :value="server" :disabled="whoisCheckStatus === 'running'">
                                <label class="form-check-label" :for="'server-' + server">{{ server }}</label>
                            </div>
                        </div>

                        <div class="jn-placeholder">
                            <p v-if="errorMsg" class="text-danger">{{ errorMsg }}</p>
                        </div>

                        <!-- 清除缓存按钮 -->
                        <div class="mt-2 mb-2 d-flex justify-content-end">
                            <button class="btn btn-sm btn-outline-secondary" @click="clearWhoisCache">
                                <i class="bi bi-trash"></i> {{ t('whois.ClearCache') || '清除缓存' }}
                            </button>
                        </div>

                        <!-- Results Table -->
                        <div v-if="whoisResults && Object.keys(whoisResults).length">

                            <div class="alert alert-success ">{{ t('whois.Note3') }}</div>
                            <div v-if="type === 'domain'" class="accordion" id="whoisResultAccordion"
                                :data-bs-theme="isDarkMode ? 'dark' : ''">
                                <div class="accordion-item" v-for="(provider, index) in providers" :key="provider">
                                    <h2 class="accordion-header" :id="'heading' + index">
                                        <button class="accordion-button" type="button" data-bs-toggle="collapse"
                                            :data-bs-target="'#collapse' + index"
                                            :aria-expanded="index === 0 ? 'true' : 'false'"
                                            :aria-controls="'collapse' + index" :class="{ collapsed: index !== 0 }">
                                            <span><i class="bi" :class="'bi-' + (index + 1) + '-circle-fill'"></i>&nbsp;
                                                <strong>{{ t('whois.Provider') }} : {{ provider.toUpperCase()
                                                    }}</strong></span>
                                        </button>
                                    </h2>
                                    <div :id="'collapse' + index" class="accordion-collapse collapse"
                                        :class="{ show: index === 0 }" :aria-labelledby="'heading' + index">
                                        <div class="accordion-body" :class="[isMobile ? ' p-2' : '']">
                                            <div class="card card-body border-0 mt-3"
                                                :class="[isDarkMode ? 'bg-black text-light' : 'bg-light']">
                                                <pre>{{ filterDomainWhoisRawData(whoisResults[providers[index]].__raw) }}</pre>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div v-else class="card card-body border-0 mt-3"
                                :class="[isDarkMode ? 'bg-black text-light' : 'bg-light']">
                                <pre>{{ filterIPWhoisRawData(whoisResults.__raw) }}</pre>
                            </div>

                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted, reactive } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';
import { trackEvent } from '@/utils/use-analytics';
import { isValidIP } from '@/utils/valid-ip.js';

const { t } = useI18n();

const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);
const isSignedIn = computed(() => store.isSignedIn);

const queryURLorIP = ref('');
const whoisCheckStatus = ref('idle');
const errorMsg = ref('');
const providers = ref([]);
const type = ref('');
const whoisResults = ref({});
const availableServers = ref([]);
const selectedServers = ref([]);

// 缓存相关
const CACHE_EXPIRY_TIME = 24 * 60 * 60 * 1000; // 24小时，单位毫秒
const whoisCache = reactive({
  data: {},

  // 从本地存储加载缓存
  loadFromStorage() {
    try {
      const cachedData = localStorage.getItem('whoisCache');
      if (cachedData) {
        const parsedData = JSON.parse(cachedData);
        this.data = parsedData;
        // 清理过期缓存
        this.cleanExpiredCache();
      }
    } catch (error) {
      console.error('Error loading whois cache:', error);
      this.data = {};
      localStorage.removeItem('whoisCache');
    }
  },

  // 保存缓存到本地存储
  saveToStorage() {
    try {
      localStorage.setItem('whoisCache', JSON.stringify(this.data));
    } catch (error) {
      console.error('Error saving whois cache:', error);
      // 如果存储失败（可能是存储空间已满），清理所有缓存
      this.clearAll();
    }
  },

  // 获取缓存数据
  get(query, servers) {
    const cacheKey = this.generateCacheKey(query, servers);
    const cachedItem = this.data[cacheKey];

    if (cachedItem && !this.isExpired(cachedItem.timestamp)) {
      console.log('Using cached whois data for:', query);
      return cachedItem.data;
    }

    return null;
  },

  // 设置缓存数据
  set(query, servers, data) {
    const cacheKey = this.generateCacheKey(query, servers);
    this.data[cacheKey] = {
      data: data,
      timestamp: Date.now()
    };
    this.saveToStorage();
  },

  // 生成缓存键
  generateCacheKey(query, servers) {
    return `${query}_${servers.sort().join(',')}`;
  },

  // 检查缓存是否过期
  isExpired(timestamp) {
    return Date.now() - timestamp > CACHE_EXPIRY_TIME;
  },

  // 清理过期缓存
  cleanExpiredCache() {
    let hasExpired = false;
    for (const key in this.data) {
      if (this.isExpired(this.data[key].timestamp)) {
        delete this.data[key];
        hasExpired = true;
      }
    }
    if (hasExpired) {
      this.saveToStorage();
    }
  },

  // 清除所有缓存
  clearAll() {
    this.data = {};
    localStorage.removeItem('whoisCache');
  }
});

// 获取 WHOIS 服务器列表
const fetchWhoisServers = async () => {
    try {
        const response = await fetch('/l-api/whois/servers');
        if (!response.ok) {
            throw new Error('Failed to fetch WHOIS servers');
        }
        const data = await response.json();
        availableServers.value = data.servers;
        // 设置默认服务器为第一个服务器
        if (availableServers.value.length > 0) {
            selectedServers.value = [availableServers.value[0]];
        }
    } catch (error) {
        console.error('Error fetching WHOIS servers:', error);
        // 设置默认服务器列表
        availableServers.value = [
            'whois.godaddy.com',
            'whois.iana.org',
            'whois.arin.net',
            'whois.apnic.net',
            'whois.ripe.net',
            'whois.lacnic.net',
            'whois.afrinic.net'
        ];
        selectedServers.value = ['whois.godaddy.com'];
    }
};

// 在组件挂载时获取服务器列表和加载缓存
onMounted(() => {
    fetchWhoisServers();
    whoisCache.loadFromStorage();
});

// 检查 URL 输入是否有效
const formatURL = (domain) => {
    // 检查是否包含协议头，若没有则尝试为其添加 http:// 以便进行 URL 格式验证
    if (!domain.match(/^https?:\/\//)) {
        domain = 'http://' + domain;
    }

    try {
        const url = new URL(domain);
        const hostname = url.hostname;

        const parts = hostname.split('.');
        const mainDomain = parts.slice(-2).join('.');

        if (mainDomain.match(/^[a-z0-9-]+(\.[a-z0-9-]+)*\.[a-z]{2,}$/i)) {
            return mainDomain;
        }
    } catch {
    }
    return false;
};

// 检查输入是否有效
const validInput = (input) => {

    if (formatURL(input)) {
        type.value = 'domain';
        return formatURL(input);
    } else if (isValidIP(input)) {
        type.value = 'ip';
        return input;
    } else {
        errorMsg.value = t('whois.invalidURL');
        return false;
    };
};

// 提交查询
const onSubmit = () => {
    trackEvent('Section', 'StartClick', 'Whois');
    errorMsg.value = '';
    providers.value = [];
    whoisResults.value = {};
    const query = validInput(queryURLorIP.value);
    if (query) {
        getWhoisResults(query);
    }
};

// 获取 Whois 结果
const getWhoisResults = async (query) => {
    whoisCheckStatus.value = 'running';

    try {
        // 检查缓存中是否有数据
        const cachedData = whoisCache.get(query, selectedServers.value);
        if (cachedData) {
            // 使用缓存数据
            if (type.value === 'domain') {
                whoisResults.value = cachedData[query];
                providers.value = Object.keys(cachedData[query]);
            } else {
                whoisResults.value = cachedData;
                providers.value = Object.keys(cachedData);
            }

            if (providers.value.length >= 1) {
                if (isSignedIn.value && query.toLowerCase().includes('ipcheck.ing')) {
                    checkAchievements();
                }
                errorMsg.value = '';
            } else {
                errorMsg.value = t('whois.fetchError');
            }

            whoisCheckStatus.value = 'idle';
            return;
        }

        // 如果缓存中没有数据，则从服务器获取
        const serverParam = selectedServers.value.join(',');
        const response = await fetch(`/l-api/whois?q=${query}&servers=${serverParam}`);

        if (!response.ok) {
            if (response.status === 429) {
                errorMsg.value = t('whois.rateLimit');
            } else {
                throw new Error('Network response was not ok');
            }
            return;
        }

        const data = await response.json();

        // 将获取的数据保存到缓存
        whoisCache.set(query, selectedServers.value, data);

        if (type.value === 'domain') {
            // 域名查询结果处理
            whoisResults.value = data[query];
            providers.value = Object.keys(data[query]);
            if (providers.value.length >= 1) {
                if (isSignedIn.value && query.toLowerCase().includes('ipcheck.ing')) {
                    checkAchievements();
                }
                errorMsg.value = '';
            } else {
                errorMsg.value = t('whois.fetchError');
            }
        } else {
            // IP 查询结果处理
            whoisResults.value = data;
            providers.value = Object.keys(data);
            if (providers.value.length >= 1) {
                errorMsg.value = '';
            } else {
                errorMsg.value = t('whois.fetchError');
            }
        }

        whoisCheckStatus.value = 'idle';
    } catch (error) {
        console.error('Error fetching Whois results:', error);
        whoisCheckStatus.value = 'idle';
        errorMsg.value = t('whois.fetchError');
    }
};

// 清除 Whois 缓存
const clearWhoisCache = () => {
    whoisCache.clearAll();
    // 显示清除成功的消息
    errorMsg.value = t('whois.CacheClearedSuccess') || '缓存已清除';
    // 2秒后清除消息
    setTimeout(() => {
        if (errorMsg.value === (t('whois.CacheClearedSuccess') || '缓存已清除')) {
            errorMsg.value = '';
        }
    }, 2000);
};

const filterDomainWhoisRawData = (text) => {
    text = text.replace(/^( {1,20})/gm, ''); // 先移除文本里，每一行开头的连续空格
    const cutoffIndex = text.indexOf('\nFor more information'); // 移除不必要的其它信息
    return cutoffIndex !== -1 ? text.substring(0, cutoffIndex) : text;
};

const filterIPWhoisRawData = (text) => {
    text = text.replace(/^#.*\n/gm, ''); // 移除所有以 # 开头的行
    text = text.replace(/^\n*/, ''); // 移除开头的所有空行
    text = text.replace(/\n$/, ''); // 移除最后一个空行
    return text;
};

// 检查是否达成成就
const checkAchievements = () => {
    if (!store.userAchievements.CuriousCat.achieved) {
        store.setTriggerUpdateAchievements('CuriousCat');
    }
}

</script>

<style scoped>
.jn-placeholder {
    height: 16pt;
}
</style>
