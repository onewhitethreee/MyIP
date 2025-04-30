<template>
    <!-- DNS Resolver -->
    <div class="dns-resolver-section my-4">
        <div class="text-secondary">
            <p>{{ t('dnsresolver.Note') }}</p>
        </div>
        <div class="row">
            <div class="col-12 mb-3">
                <div class="card jn-card" :class="{ 'dark-mode dark-mode-border': isDarkMode }">
                    <div class="card-body">
                        <div class="col-12 col-md-auto">
                            <label for="queryURL" class="col-form-label">{{ t('dnsresolver.Note2') }}</label>
                        </div>

                        <div class="input-group mb-2 mt-2">
                            <input type="text" class="form-control" :class="{ 'dark-mode': isDarkMode }"
                                :disabled="dnsCheckStatus === 'running'" :placeholder="t('dnsresolver.Placeholder')"
                                v-model="queryURL" @keyup.enter="onSubmit" name="queryURL" id="queryURL" data-1p-ignore>

                            <!-- DNS类型选择 -->
                            <button type="button" class="btn btn-primary dropdown-toggle dropdown-toggle-split"
                                data-bs-toggle="dropdown" aria-expanded="false"
                                :disabled="dnsCheckStatus === 'running' || !queryURL">
                                {{ queryType }} {{ t('dnsresolver.Record') }}
                                <span class="visually-hidden">Choose Type</span>
                            </button>
                            <ul class="dropdown-menu">
                                <li v-for="type in ['A', 'AAAA', 'CNAME', 'MX', 'NS', 'TXT']" :key="type"
                                    @click="changeType(type)">
                                    <span class="dropdown-item">{{ type }}</span>
                                </li>
                            </ul>

                            <button class="btn btn-primary" @click="onSubmit"
                                :disabled="dnsCheckStatus === 'running' || !queryURL">
                                <span v-if="dnsCheckStatus === 'idle'">{{ t('dnsresolver.Run') }}</span>
                                <span v-if="dnsCheckStatus === 'running'" class="spinner-grow spinner-grow-sm"
                                    aria-hidden="true"></span>
                            </button>
                        </div>

                        <!-- DNS服务器选择 - 单选按钮 -->
                        <div class="server-selection mt-3 mb-2">
                            <label class="form-label">{{ t('dnsresolver.SelectServer') }}:</label>
                            <div class="btn-group server-radio-group" role="group">
                                <div v-for="server in dnsServers" :key="server" class="d-inline">
                                    <input type="radio" class="btn-check" :id="`server-${server}`" name="server-options"
                                        :value="server" v-model="selectedServer"
                                        :disabled="dnsCheckStatus === 'running'">
                                    <label class="btn btn-outline-primary btn-sm"
                                        :class="{ active: selectedServer === server }" :for="`server-${server}`">
                                        {{ server }}
                                    </label>
                                </div>
                            </div>
                        </div>

                        <div class="jn-placeholder">
                            <p v-if="errorMsg" class="text-danger">{{ errorMsg }}</p>
                        </div>

                        <!-- Results Table -->
                        <div v-if="combinedResults && combinedResults.length">
                            <div class="table-responsive text-nowrap">
                                <table class="table table-hover" :class="{ 'table-dark': isDarkMode }">
                                    <thead>
                                        <tr>
                                            <th scope="col">{{ t('dnsresolver.Provider') }}</th>
                                            <th scope="col">{{ t('dnsresolver.Result') }}</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <tr v-for="(result, index) in combinedResults" :key="index">
                                            <td>{{ result.provider }}</td>
                                            <td>
                                                <div
                                                    v-if="Array.isArray(result.addresses) && result.addresses.length > 0">
                                                    <div v-for="(address, addrIndex) in result.addresses"
                                                        :key="addrIndex" :class="{ 'opacity-50': address === 'N/A' }">
                                                        {{ address }}
                                                    </div>
                                                </div>
                                                <div v-else :class="{ 'opacity-50': result.addresses === 'N/A' }">
                                                    {{ result.addresses }}
                                                </div>
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
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';
import { trackEvent } from '@/utils/use-analytics';

const { t } = useI18n();

const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);

const queryURL = ref('');
const queryType = ref('A');
const dnsCheckStatus = ref('idle');
const errorMsg = ref('');
const combinedResults = ref([]);

// DNS服务器列表和选中的服务器
const dnsServers = ref([
    'Google',
    'Cloudflare',
    'OpenDNS',
    'Quad9',
    'ControlD',
    'AdGuard',
    'Quad 101',
    'AliDNS',
    'DNSPod',
    '114DNS',
    'China Unicom'
]);
// 默认选择Google
const selectedServer = ref('Google');

// 组件加载时设置默认服务器
onMounted(() => {
    selectedServer.value = 'Google';
});

// 检查 URL 输入是否有效
const validateInput = (input) => {
    // 检查是否包含协议头，若没有则尝试为其添加 http:// 以便进行 URL 格式验证
    if (!input.match(/^https?:\/\//)) {
        input = 'http://' + input;
    }

    try {
        const url = new URL(input);
        const hostname = url.hostname;

        if (hostname.match(/^[a-z0-9-]+(\.[a-z0-9-]+)*\.[a-z]{2,}$/i)) {
            return hostname;
        }
    } catch {
    }

    errorMsg.value = t('dnsresolver.invalidURL');
    return null;
};

const changeType = (type) => {
    queryType.value = type;
};

const onSubmit = () => {
    trackEvent('Section', 'StartClick', 'DNSResolver');
    errorMsg.value = '';

    const hostname = validateInput(queryURL.value);
    const type = queryType.value;

    if (hostname) {
        getDNSResults(hostname, type, selectedServer.value);
    }
};

// 获取DNS结果
const getDNSResults = async (hostname, type, server) => {
    combinedResults.value = [];
    dnsCheckStatus.value = 'running';
    try {
        const response = await fetch(`/l-api/dnsresolver?hostname=${hostname}&type=${type}&server=${server}`);
        if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.error || 'Network response was not ok');
        }
        const data = await response.json();
        processResults(data);
        dnsCheckStatus.value = 'idle';
        errorMsg.value = '';
    } catch (error) {
        console.error('Error fetching DNS results:', error);
        dnsCheckStatus.value = 'idle';
        errorMsg.value = error.message || t('dnsresolver.fetchError');
    }
};

// 修改处理结果函数以支持多条记录
const processResults = (data) => {
    const processEntries = (entries, type) => entries.map(entry => {
        const provider = Object.keys(entry)[0];
        const addresses = entry[provider];

        return {
            provider: `${provider} (${type})`,
            addresses: addresses
        };
    });

    if (data.result_dns) {
        combinedResults.value.push(...processEntries(data.result_dns, 'DNS'));
    }
    if (data.result_doh) {
        combinedResults.value.push(...processEntries(data.result_doh, 'DoH 🔒'));
    }
};
</script>

<style scoped>
.jn-placeholder {
    height: 16pt;
}

.server-selection {
    padding-top: 0.5rem;
    border-top: 1px solid #dee2e6;
}

.server-radio-group {
    display: flex;
    flex-wrap: wrap;
    gap: 0.25rem;
    margin-top: 0.5rem;
}

/* 响应式调整 */
@media (max-width: 768px) {
    .server-radio-group {
        gap: 0.15rem;
    }

    .server-radio-group .btn {
        padding: 0.25rem 0.5rem;
        font-size: 0.75rem;
    }
}

/* 暗黑模式边框 */
:deep(.dark-mode) .server-radio-group .btn-outline-primary {
    border-color: #6c757d;
    color: #f8f9fa;
}

:deep(.dark-mode) .server-radio-group .btn-outline-primary.active {
    background-color: #0d6efd;
    color: #fff;
}
</style>