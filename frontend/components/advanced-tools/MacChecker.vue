<template>
    <!-- MAC Checker -->
    <div class="mac-checker-section my-4">
        <div class="text-secondary">
            <p>{{ t('macchecker.Note') }}</p>
        </div>
        <div class="row">
            <div class="col-12 mb-3">
                <div class="card jn-card" :class="{ 'dark-mode dark-mode-border': isDarkMode }">
                    <div class="card-body">
                        <div class="col-12 col-md-auto">
                            <label for="queryMAC" class="col-form-label">{{ t('macchecker.Note2') }}</label>
                        </div>

                        <div class="input-group mb-2 mt-2">
                            <input type="text" class="form-control" :class="{ 'dark-mode': isDarkMode }"
                                :disabled="macCheckStatus === 'running'" :placeholder="t('macchecker.Placeholder')"
                                v-model="queryMAC" @keyup.enter="onSubmit" name="queryMAC" id="queryMAC" data-1p-ignore>

                            <button class="btn btn-primary" @click="onSubmit"
                                :disabled="macCheckStatus === 'running' || !queryMAC">
                                <span v-if="macCheckStatus !== 'running'">{{ t('macchecker.Run') }}</span>
                                <span v-else class="spinner-grow spinner-grow-sm" aria-hidden="true"></span>
                            </button>
                        </div>

                        <!-- 添加服务器选择 -->
                        <div class="mb-3">
                            <label class="form-label">{{ t('macchecker.SelectServers') }}</label>
                            <div class="server-options">
                                <div class="form-check" v-for="server in availableServers" :key="server">
                                    <input 
                                        type="radio" 
                                        class="form-check-input" 
                                        :class="{
                                            'jn-check-dark': isDarkMode,
                                            'jn-check-light': !isDarkMode
                                        }"
                                        :name="'server_' + server"
                                        :id="'server_' + server"
                                        :value="server"
                                        v-model="selectedServer">
                                    <label class="form-check-label jn-number" :for="'server_' + server">
                                        {{ server }}
                                    </label>
                                </div>
                            </div>
                        </div>

                        <!-- 添加清除缓存按钮 -->
                        <div class="mt-2 mb-2 d-flex justify-content-end">
                            <button class="btn btn-sm btn-outline-secondary" @click="clearCache">
                                <i class="bi bi-trash"></i> {{ t('macchecker.ClearCache') || '清除缓存' }}
                            </button>
                        </div>

                        <div class="jn-placeholder">
                            <p v-if="errorMsg" class="text-danger">{{ errorMsg }}</p>
                        </div>

                        <!-- Result Display -->

                        <div id="macCheckResult" class="row" v-if="macCheckResult.success">
                            <div class="col-lg-8 col-md-8 col-12 mb-4">
                                <div class="card h-100" :class="{ 'dark-mode dark-mode-border': isDarkMode }">
                                    <div class="card-body row">
                                        <h3 class="mb-4">{{ t('macchecker.manufacturer') }}</h3>
                                        <div class="col-lg-6 col-md-6 col-12">
                                            <div class="jn-detail" v-for="item in leftItems" :key="item.key">
                                                <span>
                                                    {{ t(`macchecker.${item.key}`) }}
                                                </span>
                                                <span class="jn-con-title card-title mt-1">
                                                    {{ macCheckResult[item.key] }}
                                                </span>
                                            </div>
                                        </div>

                                        <div class="col-lg-6 col-md-6 col-12">
                                            <div class="jn-detail">
                                                <span>
                                                    {{ t('macchecker.company') }}
                                                </span>
                                                <span class="jn-con-title card-title mt-1">
                                                    {{ macCheckResult.company }}
                                                </span>
                                            </div>
                                            <div v-if="macCheckResult.country !== 'N/A'" class="jn-detail">
                                                <span>
                                                    {{ t('macchecker.country') }}
                                                </span>
                                                <span class="jn-con-title card-title mt-1">
                                                    <span
                                                        :class="'jn-fl fi fi-' + macCheckResult.country.toLowerCase()"></span>
                                                    {{ getCountryName(macCheckResult.country, lang) }}
                                                </span>
                                            </div>
                                            <div class="jn-detail">
                                                <span>
                                                    {{ t('macchecker.address') }}
                                                </span>
                                                <span class="jn-con-title card-title mt-1">
                                                    {{ macCheckResult.address }}
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div class="col-lg-4 col-md-4 col-12 mb-4">
                                <div class="card h-100" :class="{ 'dark-mode dark-mode-border': isDarkMode}">
                                    <div class="card-body">
                                        <h3 class="mb-4">{{ t('macchecker.property') }}</h3>
                                        <div class="table-responsive text-nowrap">
                                            <table class="table table-hover" :class="{ 'table-dark': isDarkMode }">
                                                <thead>
                                                    <tr>
                                                        <th scope="col">{{ t('macchecker.property') }}</th>
                                                        <th scope="col">{{ t('macchecker.value') }}</th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    <tr v-for="item in tableItems" :key="item.key">
                                                        <td>{{ t(`macchecker.${item.key}`) }}</td>
                                                        <td>
                                                            <i class="bi"
                                                                :class="macCheckResult[item.key] ? 'bi-check-circle-fill text-success' : 'bi-x-circle-fill text-secondary'"></i>
                                                        </td>
                                                    </tr>
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            </div>

                        </div>

                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';
import { trackEvent } from '@/utils/use-analytics';
import getCountryName from '@/utils/country-name.js';

const { t } = useI18n();

const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);
const lang = computed(() => store.lang);

const macCheckResult = ref({});
const macCheckStatus = ref("idle");
const queryMAC = ref('');
const errorMsg = ref('');

// 缓存相关常量
const CACHE_KEY = 'mac_lookup_cache';
const CACHE_EXPIRY = 24 * 60 * 60 * 1000; // 24小时

const availableServers = ref(['api.maclookup.app/v2/macs', 'api.macvendors.com']);
const selectedServer = ref('api.maclookup.app/v2/macs'); // 默认服务器

// 缓存管理
const cache = {
    get(mac, server) {
        try {
            const cached = localStorage.getItem(CACHE_KEY);
            if (!cached) return null;

            const cacheData = JSON.parse(cached);
            const cacheKey = `${mac}_${server}`;
            const item = cacheData[cacheKey];

            if (!item) return null;
            if (Date.now() - item.timestamp > CACHE_EXPIRY) {
                delete cacheData[cacheKey];
                localStorage.setItem(CACHE_KEY, JSON.stringify(cacheData));
                return null;
            }

            return item.data;
        } catch (error) {
            console.error('Cache read error:', error);
            return null;
        }
    },

    set(mac, server, data) {
        try {
            const cached = localStorage.getItem(CACHE_KEY);
            const cacheData = cached ? JSON.parse(cached) : {};
            const cacheKey = `${mac}_${server}`;

            cacheData[cacheKey] = {
                data,
                timestamp: Date.now()
            };

            localStorage.setItem(CACHE_KEY, JSON.stringify(cacheData));
        } catch (error) {
            console.error('Cache write error:', error);
        }
    },

    clear() {
        try {
            localStorage.removeItem(CACHE_KEY);
        } catch (error) {
            console.error('Cache clear error:', error);
        }
    }
};

// 清除缓存
const clearCache = () => {
    cache.clear();
    errorMsg.value = t('macchecker.CacheCleared') || '缓存已清除';
    setTimeout(() => {
        errorMsg.value = '';
    }, 2000);
};

const leftItems = computed(() => {
    return [
        { key: 'macPrefix' },
        { key: 'blockStart' },
        { key: 'blockEnd' },
        { key: 'blockSize' },
        { key: 'blockType' }
    ];
});

const tableItems = computed(() => {
    return [
        { key: 'isRand' },
        { key: 'isPrivate' },
        { key: 'isMulticast' },
        { key: 'isUnicast' },
        { key: 'isLocal' },
        { key: 'isGlobal' }
    ];
});

// 获取 MAC 信息的函数
const getMacInfo = async (mac) => {
    if (!mac) return;
    
    // 检查缓存
    const cachedResult = cache.get(mac, selectedServer.value);
    if (cachedResult) {
        macCheckResult.value = cachedResult;
        macCheckResult.value.success = true;
        return;
    }

    macCheckStatus.value = 'running';
    errorMsg.value = '';
    
    try {
        const response = await fetch(`/l-api/mac?q=${encodeURIComponent(mac)}&servers=${selectedServer.value}`);
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        const data = await response.json();
        
        if (data.error) {
            errorMsg.value = data.error;
            macCheckResult.value = {};
        } else {
            const result = data[selectedServer.value] || {};
            macCheckResult.value = result;
            macCheckResult.value.success = true;
            
            // 保存到缓存
            if (Object.keys(result).length > 0) {
                cache.set(mac, selectedServer.value, result);
            }
        }
    } catch (error) {
        console.error('Error:', error);
        errorMsg.value = t('macchecker.fetchError');
        macCheckResult.value = {};
    } finally {
        macCheckStatus.value = 'idle';
    }
};

// 提交查询
const onSubmit = () => {
    if (!queryMAC.value || macCheckStatus.value === 'running') return;
    getMacInfo(queryMAC.value);
};

// 获取服务器列表并设置默认值
const fetchServers = async () => {
    try {
        const response = await fetch('/l-api/mac/servers');
        if (!response.ok) {
            throw new Error('Failed to fetch servers');
        }
        const data = await response.json();
        availableServers.value = data.servers || availableServers.value;
        // 确保设置默认服务器
        if (availableServers.value.length > 0 && !selectedServer.value) {
            selectedServer.value = availableServers.value[0];
        }
    } catch (error) {
        console.error('Error fetching servers:', error);
        errorMsg.value = t('macchecker.serversFetchError');
        // 如果获取服务器列表失败，使用默认值
        if (!selectedServer.value && availableServers.value.length > 0) {
            selectedServer.value = availableServers.value[0];
        }
    }
};

// 在组件挂载时获取服务器列表
onMounted(() => {
    fetchServers();
});
</script>

<style scoped>
.jn-placeholder {
    height: 16pt;
}

.jn-detail {
    display: flex;
    flex-direction: column;
    align-content: flex-start;
    align-items: flex-start;
    flex-wrap: wrap;
    margin-bottom: 10pt;
}

.server-options {
    margin-bottom: 1rem;
}

.jn-number {
    font-family: var(--bs-font-monospace);
    font-size: 0.9rem;
}
</style>
