<template>
    <!-- Enhanced MAC Checker -->
    <div class="mac-checker-section my-5">
        <div class="text-secondary mb-3">
            <p class="lead">{{ t('macchecker.Note') }}</p>
        </div>
        <div class="row">
            <div class="col-12 mb-4">
                <div class="card shadow-sm rounded-lg" :class="{ 'dark-mode dark-mode-border': isDarkMode }">
                    <div class="card-header bg-transparent py-3">
                        <h2 class="card-title mb-0 fw-bold">
                            <i class="bi bi-wifi me-2"></i>{{ t('macchecker.Title') || 'MAC Address Checker' }}
                        </h2>
                    </div>
                    <div class="card-body p-4">
                        <div class="col-12">
                            <label for="queryMAC" class="form-label fw-semibold">
                                {{ t('macchecker.Note2') }}
                            </label>
                        </div>

                        <div class="input-group mb-3">
                            <span class="input-group-text bg-light border-end-0"
                                :class="{ 'bg-dark text-light': isDarkMode }">
                                <i class="bi bi-search"></i>
                            </span>
                            <input type="text" class="form-control border-start-0 shadow-none"
                                :class="{ 'dark-mode': isDarkMode }" :disabled="macCheckStatus === 'running'"
                                :placeholder="t('macchecker.Placeholder')" v-model="queryMAC" @keyup.enter="onSubmit"
                                name="queryMAC" id="queryMAC" data-1p-ignore>

                            <button class="btn btn-primary px-4" @click="onSubmit"
                                :disabled="macCheckStatus === 'running' || !queryMAC">
                                <span v-if="macCheckStatus !== 'running'">
                                    <i class="bi bi-search me-1"></i>{{ t('macchecker.Run') }}
                                </span>
                                <span v-else>
                                    <span class="spinner-grow spinner-grow-sm me-1" aria-hidden="true"></span>
                                    {{ t('macchecker.Searching') || 'Searching...' }}
                                </span>
                            </button>
                        </div>

                        <!-- Server selection -->
                        <div class="mb-4 p-3 bg-light rounded" :class="{ 'bg-dark': isDarkMode }">
                            <label class="form-label fw-semibold mb-2">
                                <i class="bi bi-server me-1"></i>{{ t('macchecker.SelectServers') }}
                            </label>
                            <div class="server-options d-flex flex-wrap gap-3">
                                <div class="form-check" v-for="server in availableServers" :key="server">
                                    <input type="radio" class="form-check-input" :class="{
                                        'jn-check-dark': isDarkMode,
                                        'jn-check-light': !isDarkMode
                                    }" :name="'server_' + server" :id="'server_' + server" :value="server"
                                        v-model="selectedServer">
                                    <label class="form-check-label jn-number" :for="'server_' + server">
                                        {{ server }}
                                    </label>
                                </div>
                            </div>

                            <!-- Cache clear button -->
                            <div class="mt-3 d-flex justify-content-end">
                                <button class="btn btn-sm btn-outline-secondary" @click="clearCache">
                                    <i class="bi bi-trash me-1"></i> {{ t('macchecker.ClearCache') || 'Clear Cache' }}
                                </button>
                            </div>
                        </div>

                        <!-- Error message display -->
                        <div class="jn-placeholder">
                            <div v-if="errorMsg" class="alert alert-danger py-2 px-3 d-flex align-items-center">
                                <i class="bi bi-exclamation-triangle-fill me-2"></i>
                                <span>{{ errorMsg }}</span>
                            </div>
                        </div>

                        <!-- Results Display -->
                        <div id="macCheckResult" class="row mt-4" v-if="macCheckResult.success">
                            <div class="col-lg-8 col-md-8 col-12 mb-4">
                                <div class="card h-100 border-primary border-opacity-25 shadow-sm"
                                    :class="{ 'dark-mode dark-mode-border': isDarkMode }">
                                    <div class="card-header bg-primary bg-opacity-10 py-3">
                                        <h3 class="mb-0 fs-4">
                                            <i class="bi bi-building me-2"></i>{{ t('macchecker.manufacturer') }}
                                        </h3>
                                    </div>
                                    <div class="card-body p-4 row">
                                        <div class="col-lg-6 col-md-6 col-12">
                                            <div class="jn-detail mb-3" v-for="item in leftItems" :key="item.key">
                                                <span class="text-secondary small text-uppercase">
                                                    {{ t(`macchecker.${item.key}`) }}
                                                </span>
                                                <span class="jn-con-title fs-5 fw-semibold mt-1 text-primary">
                                                    {{ macCheckResult[item.key] }}
                                                </span>
                                            </div>
                                        </div>

                                        <div class="col-lg-6 col-md-6 col-12">
                                            <div class="jn-detail mb-3">
                                                <span class="text-secondary small text-uppercase">
                                                    {{ t('macchecker.company') }}
                                                </span>
                                                <span class="jn-con-title fs-5 fw-semibold mt-1 text-primary">
                                                    {{ macCheckResult.company }}
                                                </span>
                                            </div>
                                            <div v-if="macCheckResult.country !== 'N/A'" class="jn-detail mb-3">
                                                <span class="text-secondary small text-uppercase">
                                                    {{ t('macchecker.country') }}
                                                </span>
                                                <span
                                                    class="jn-con-title fs-5 fw-semibold mt-1 d-flex align-items-center">
                                                    <span
                                                        :class="'jn-fl fi fi-' + macCheckResult.country.toLowerCase() + ' me-2'"></span>
                                                    {{ getCountryName(macCheckResult.country, lang) }}
                                                </span>
                                            </div>
                                            <div class="jn-detail mb-3">
                                                <span class="text-secondary small text-uppercase">
                                                    {{ t('macchecker.address') }}
                                                </span>
                                                <span class="jn-con-title fs-5 fw-semibold mt-1">
                                                    {{ macCheckResult.address }}
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div class="col-lg-4 col-md-4 col-12 mb-4">
                                <div class="card h-100 border-success border-opacity-25 shadow-sm"
                                    :class="{ 'dark-mode dark-mode-border': isDarkMode }">
                                    <div class="card-header bg-success bg-opacity-10 py-3">
                                        <h3 class="mb-0 fs-4">
                                            <i class="bi bi-info-circle me-2"></i>{{ t('macchecker.property') }}
                                        </h3>
                                    </div>
                                    <div class="card-body p-0">
                                        <div class="table-responsive">
                                            <table class="table table-hover mb-0" :class="{ 'table-dark': isDarkMode }">
                                                <thead>
                                                    <tr class="table-light" :class="{ 'table-secondary': isDarkMode }">
                                                        <th scope="col" class="border-0">{{ t('macchecker.property') }}
                                                        </th>
                                                        <th scope="col" class="border-0 text-center">{{
                                                            t('macchecker.value') }}</th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    <tr v-for="item in tableItems" :key="item.key">
                                                        <td class="fw-medium">{{ t(`macchecker.${item.key}`) }}</td>
                                                        <td class="text-center">
                                                            <i class="bi" :class="[
                                                                macCheckResult[item.key]
                                                                    ? 'bi-check-circle-fill text-success fs-5'
                                                                    : 'bi-x-circle-fill text-secondary fs-5'
                                                            ]"></i>
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
import { ref, computed, onMounted, watch } from 'vue';
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

// Cache related constants
const CACHE_KEY = 'mac_lookup_cache';
const CACHE_EXPIRY = 24 * 60 * 60 * 1000; // 24 hours

const availableServers = ref(['api.maclookup.app/v2/macs', 'api.macvendors.com']);
const selectedServer = ref('api.maclookup.app/v2/macs'); // Default server

// Watch for server changes to clear the result
watch(selectedServer, () => {
    if (Object.keys(macCheckResult.value).length > 0) {
        macCheckResult.value = {};
        // Optional: If you want to automatically search with the new server
        if (queryMAC.value) {
            getMacInfo(queryMAC.value);
        }
    }
});

// Cache management
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

// Clear cache
const clearCache = () => {
    cache.clear();
    errorMsg.value = t('macchecker.CacheCleared') || 'Cache has been cleared';

    // Fade out error message after 2 seconds
    setTimeout(() => {
        errorMsg.value = '';
    }, 2000);

    // Track event
    trackEvent('mac_checker', 'clear_cache');
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

// Function to get MAC information
const getMacInfo = async (mac) => {
    if (!mac) return;

    // Format MAC address (optional)
    const formattedMac = mac.trim();

    // Check cache
    const cachedResult = cache.get(formattedMac, selectedServer.value);
    if (cachedResult) {
        macCheckResult.value = cachedResult;
        macCheckResult.value.success = true;

        // Track event - cache hit
        trackEvent('mac_checker', 'cache_hit', selectedServer.value);
        return;
    }

    macCheckStatus.value = 'running';
    errorMsg.value = '';

    try {
        // Track event - search started
        trackEvent('mac_checker', 'search', selectedServer.value);

        const response = await fetch(`/l-api/mac?q=${encodeURIComponent(formattedMac)}&servers=${selectedServer.value}`);
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        const data = await response.json();

        if (data.error) {
            errorMsg.value = data.error;
            macCheckResult.value = {};

            // Track event - search error
            trackEvent('mac_checker', 'search_error', data.error);
        } else {
            const result = data[selectedServer.value] || {};
            macCheckResult.value = result;
            macCheckResult.value.success = true;

            // Save to cache
            if (Object.keys(result).length > 0) {
                cache.set(formattedMac, selectedServer.value, result);

                // Track event - search success
                trackEvent('mac_checker', 'search_success', selectedServer.value);
            }
        }
    } catch (error) {
        console.error('Error:', error);
        errorMsg.value = t('macchecker.fetchError') || 'Failed to fetch data. Please try again.';
        macCheckResult.value = {};

        // Track event - search exception
        trackEvent('mac_checker', 'search_exception', error.message);
    } finally {
        macCheckStatus.value = 'idle';
    }
};

// Submit query
const onSubmit = () => {
    if (!queryMAC.value || macCheckStatus.value === 'running') return;
    getMacInfo(queryMAC.value);
};

// Get server list and set default value
const fetchServers = async () => {
    try {
        const response = await fetch('/l-api/mac/servers');
        if (!response.ok) {
            throw new Error('Failed to fetch servers');
        }
        const data = await response.json();
        availableServers.value = data.servers || availableServers.value;
        // Ensure default server is set
        if (availableServers.value.length > 0 && !selectedServer.value) {
            selectedServer.value = availableServers.value[0];
        }
    } catch (error) {
        console.error('Error fetching servers:', error);
        errorMsg.value = t('macchecker.serversFetchError') || 'Failed to fetch server list';
        // Use default value if server list fetch fails
        if (!selectedServer.value && availableServers.value.length > 0) {
            selectedServer.value = availableServers.value[0];
        }
    }
};

// Get servers on component mount
onMounted(() => {
    fetchServers();
});
</script>

<style scoped>
.mac-checker-section {
    font-family: var(--bs-font-sans-serif);
}

.jn-placeholder {
    min-height: 50px;
    transition: all 0.3s ease;
}

.jn-detail {
    display: flex;
    flex-direction: column;
    align-content: flex-start;
    align-items: flex-start;
    flex-wrap: wrap;
    padding: 0.5rem 0;
    transition: all 0.2s ease;
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.dark-mode .jn-detail {
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.jn-detail:last-child {
    border-bottom: none;
}

.jn-detail:hover {
    background-color: rgba(0, 0, 0, 0.01);
}

.dark-mode .jn-detail:hover {
    background-color: rgba(255, 255, 255, 0.02);
}

.server-options {
    margin-bottom: 1rem;
    transition: all 0.3s ease;
}

.jn-number {
    font-family: var(--bs-font-monospace);
    font-size: 0.9rem;
}

.card {
    transition: all 0.3s ease;
    border-radius: 0.5rem;
    overflow: hidden;
}

.card-header {
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.dark-mode .card-header {
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

/* Animation for loading state */
.spinner-grow {
    animation: spinner-grow 0.75s linear infinite;
}

/* Responsive adjustments */
@media (max-width: 768px) {
    .card-body {
        padding: 1rem;
    }

    .jn-con-title {
        font-size: 1rem !important;
    }
}

/* Flag icon enhancement */
.jn-fl {
    display: inline-block;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    border-radius: 2px;
    width: 24px;
    height: 18px;
    background-size: cover;
}
</style>