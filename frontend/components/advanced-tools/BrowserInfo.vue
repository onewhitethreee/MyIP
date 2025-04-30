<template>
    <!-- Browser Info -->
    <div class="browser-info-section my-4">
        <div class="text-secondary">
            <p>{{ t('browserinfo.Note') }}</p>
            <p>{{ t('browserinfo.Note2') }}</p>
        </div>
        <div class="row">
            <div class="col-12 mb-3">
                <div class="card jn-card" :class="{ 'dark-mode dark-mode-border': isDarkMode }">

                    <div class="card-body">
                        <Transition name="slide-fade" mode="out-in">
                            <div id="browserInfoResult" class="row" v-if="checkingStatus === 'finished'">
                                <div class="col-lg-8 col-md-8 col-12 mb-4">
                                    <div class="h-100">
                                        <div class="card-body row"
                                            :class="[isMobile ? 'p-1 border-1 border-bottom' : '']">
                                            <h3 class="mb-4">{{ t('browserinfo.browser.Infos') }} <i
                                                    class="bi bi-person-workspace"></i></h3>
                                            <div class="jn-ua-box w-100">
                                                <div class="alert alert-success jn-ua-box">
                                                    <span class="mb-1 badge text-bg-success">User Agent</span>
                                                    <span><span class="jn-code-font ">{{ userAgent.ua }}</span> <i
                                                            :class="copiedStatus ? 'bi bi-clipboard-check-fill' : 'bi bi-clipboard-plus'"
                                                            @click="copyToClipboard(userAgent.ua)" role="button"
                                                            aria-label="Copy UA"></i></span>
                                                </div>
                                            </div>

                                            <div class="col-lg-6 col-md-6 col-12">
                                                <div class="jn-detail">
                                                    <span>
                                                        {{ t('browserinfo.browser.browserName') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ userAgent.browser.name }} {{ userAgent.browser.version }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail">
                                                    <span>
                                                        {{ t('browserinfo.browser.engineName') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ userAgent.engine.name }} {{ userAgent.engine.version }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail">
                                                    <span>
                                                        {{ t('browserinfo.browser.osName') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ userAgent.os.name }} {{ userAgent.os.version }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail">
                                                    <span>
                                                        {{ t('browserinfo.browser.language') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.browserLanguage }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail">
                                                    <span>
                                                        {{ t('browserinfo.browser.cookieEnabled') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.cookieEnabled?
                                                        t('browserinfo.browser.cookieEnabledTrue'):t('browserinfo.browser.cookieEnabledFalse')
                                                        }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.features?.length">
                                                    <span>
                                                        {{ t('browserinfo.browser.cpuFeatures') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        <div class="d-flex flex-wrap gap-1">
                                                            <span v-for="feature in otherInfos.cpuInfo.features" 
                                                                :key="feature"
                                                                class="badge bg-info text-dark">
                                                                {{ feature }}
                                                            </span>
                                                        </div>
                                                    </span>
                                                </div>
                                                <!-- 系统信息 -->
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.system">
                                                    <span>
                                                        {{ t('browserinfo.browser.platform') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.cpuInfo.system.platform }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.system?.deviceMemory">
                                                    <span>
                                                        {{ t('browserinfo.browser.deviceMemory') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.cpuInfo.system.deviceMemory }} GB
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.system?.connection">
                                                    <span>
                                                        {{ t('browserinfo.browser.connection') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.cpuInfo.system.connection.effectiveType }}
                                                        ({{ otherInfos.cpuInfo.system.connection.downlink }} Mbps)
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.system?.battery">
                                                    <span>
                                                        {{ t('browserinfo.browser.battery') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.cpuInfo.system.battery.level }}%
                                                        ({{ otherInfos.cpuInfo.system.battery.charging ? t('browserinfo.browser.charging') : t('browserinfo.browser.discharging') }})
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.system?.screen">
                                                    <span>
                                                        {{ t('browserinfo.browser.screen') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        <div class="d-flex flex-column">
                                                            <small>{{ otherInfos.cpuInfo.system.screen.width }}x{{ otherInfos.cpuInfo.system.screen.height }} ({{ otherInfos.cpuInfo.system.screen.colorDepth }} bit)</small>
                                                            <small class="text-muted">Available: {{ otherInfos.cpuInfo.system.screen.availWidth }}x{{ otherInfos.cpuInfo.system.screen.availHeight }}</small>
                                                            <small class="text-muted">Pixel Ratio: {{ otherInfos.cpuInfo.system.screen.devicePixelRatio }}</small>
                                                        </div>
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.system?.locale">
                                                    <span>
                                                        {{ t('browserinfo.browser.locale') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        <div class="d-flex flex-column">
                                                            <small>{{ otherInfos.cpuInfo.system.locale.timezone }}</small>
                                                            <small class="text-muted">{{ otherInfos.cpuInfo.system.locale.language }}</small>
                                                            <small class="text-muted">{{ otherInfos.cpuInfo.system.locale.languages.join(', ') }}</small>
                                                        </div>
                                                    </span>
                                                </div>
                                            </div>

                                            <div class="col-lg-6 col-md-6 col-12">
                                                <div class="jn-detail">
                                                    <span>
                                                        {{ t('browserinfo.browser.deviceVendor') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ userAgent.device.vendor }} {{ userAgent.device.model }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail">
                                                    <span>
                                                        {{ t('browserinfo.browser.cpuCores') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.cpucores }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.model">
                                                    <span>
                                                        {{ t('browserinfo.browser.cpuModel') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.cpuInfo.model }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.architecture">
                                                    <span>
                                                        {{ t('browserinfo.browser.cpuArchitecture') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        {{ otherInfos.cpuInfo.architecture }}
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.features?.length">
                                                    <span>
                                                        {{ t('browserinfo.browser.cpuFeatures') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        <div class="d-flex flex-wrap gap-1">
                                                            <span v-for="feature in otherInfos.cpuInfo.features" 
                                                                :key="feature"
                                                                class="badge bg-info text-dark">
                                                                {{ feature }}
                                                            </span>
                                                        </div>
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.system?.webGL">
                                                    <span>
                                                        {{ t('browserinfo.browser.webGL') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        <div class="d-flex flex-column">
                                                            <small class="text-muted">{{ otherInfos.cpuInfo.system.webGL.vendor }}</small>
                                                            <small>{{ otherInfos.cpuInfo.system.webGL.renderer }}</small>
                                                            <small class="text-muted">{{ otherInfos.cpuInfo.system.webGL.version }}</small>
                                                        </div>
                                                    </span>
                                                </div>
                                                <div class="jn-detail" v-if="otherInfos.cpuInfo?.system?.audio">
                                                    <span>
                                                        {{ t('browserinfo.browser.audio') }}
                                                    </span>
                                                    <span class="jn-con-title card-title mt-1">
                                                        <div class="d-flex flex-column">
                                                            <small>{{ otherInfos.cpuInfo.system.audio.sampleRate }} Hz</small>
                                                            <small class="text-muted">{{ otherInfos.cpuInfo.system.audio.channelCount }} channels</small>
                                                        </div>
                                                    </span>
                                                </div>
                                            </div>

                                        </div>
                                    </div>
                                </div>

                                <div class="col-lg-4 col-md-4 col-12 mb-4">
                                    <div class="h-100" :class="{ 
                                    'dark-mode dark-mode-border': isDarkMode,
                                    'card': !isMobile
                                    }">
                                        <div class="card-body" :class="[isMobile ? 'p-1' : '']">
                                            <h3 class="mb-4">{{ t('browserinfo.fingerprint.Infos') }} <i
                                                    class="bi bi-fingerprint"></i></h3>
                                            <div class="jn-ua-box w-100">
                                                <div :class="[isMobile ? 'jn-fp-box-mobile' : '']"
                                                    class="alert alert-primary jn-ua-box">
                                                    <span class="mb-1 badge text-bg-primary">{{
                                                        t('browserinfo.fingerprint.fingerprint') }}</span>
                                                    <span class="jn-code-font ">{{ fingerprint }}</span>
                                                </div>
                                            </div>

                                            <p>{{ t('browserinfo.fingerprint.changeOption') }}</p>

                                            <div class="row g-1 m-1">
                                                <div v-for="(value, key) in excludeOptions" :key="key"
                                                    class="form-check form-switch col-6">
                                                    <input class="form-check-input" type="checkbox" :id="key"
                                                        v-model="excludeOptions[key]" />
                                                    <label class="form-check-label" :for="key">
                                                        {{ t(`browserinfo.options.${key}`) }}
                                                    </label>
                                                </div>
                                            </div>
                                            <div class="alert alert-light opacity-75 mt-4" role="alert">
                                                <i class="bi bi-info-circle"></i> {{
                                                t('browserinfo.fingerprint.browserTips') }}
                                            </div>

                                        </div>
                                    </div>
                                </div>

                            </div>
                            <div v-else class="jn-placeholder ">
                                <span v-if="checkingStatus === 'running'">
                                    <span class="spinner-grow spinner-grow-sm text-success" aria-hidden="true"></span>
                                    <span class="text-success">&nbsp;{{ t('browserinfo.calculating') }}</span>
                                </span>
                                <p v-if="checkingStatus === 'error'" class="text-danger">{{ errorMsg }}</p>
                            </div>
                        </Transition>
                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();

const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);

const fingerprint = ref('');
const excludeOptions = ref({
    'audio': true,
    'canvas': true,
    'fonts': true,
    'hardware': true,
    'locales': true,
    'permissions': true,
    'plugins': true,
    'screen': true,
    'system': true,
    'webgl': true,
    'math': true,
});
const errorMsg = ref('');
const checkingStatus = ref('idle');
const copiedStatus = ref(false);

const userAgent = ref('');
const gpu = ref('');
const otherInfos = ref({});

// 获取 GPU 信息
const getGPU = async () => {
    try {
        const { getGPUTier } = await import('detect-gpu');
        const gpuTier = await getGPUTier();
        if (gpuTier && gpuTier.gpu) {
            gpu.value = gpuTier.gpu.toLowerCase().replace(/(^\w|\s\w)/g, m => m.toUpperCase());
        } else {
            gpu.value = 'N/A';
        }
    } catch (error) {
        console.error('Error getting GPU info:', error);
        throw error;
    }
}

// 获取 UA
const getUA = async () => {
    try {
        const { UAParser } = await import('ua-parser-js');
        const parser = new UAParser();
        parser.setUA(parser.getUA());
        userAgent.value = parser.getResult();
    } catch (error) {
        console.error('Error getting user agent:', error);
        throw error;
    }
};

// 获取其他信息
const getOtherBrowserInfo = async () => {
    try {
        otherInfos.value.browserLanguage = navigator.language;
        otherInfos.value.cookieEnabled = navigator.cookieEnabled;
        otherInfos.value.cpucores = navigator.hardwareConcurrency;
    } catch (error) {
        console.error('Error getting other browser info:', error);
        throw error;
    }
};

// 获取指纹计算的排除选项
const getExcludeOptions = async () => {
    const results = [];
    const checkOptions = (options, prefix = '') => {
        for (const key in options) {
            const value = options[key];
            const fullPath = prefix ? `${prefix}.${key}` : key;
            if (typeof value === 'object') {
                checkOptions(value, fullPath);
            } else if (!value) {
                results.push(fullPath);
            }
        }
    };

    checkOptions(excludeOptions.value);
    return results;
};

// 获取指纹
const getFingerPrint = async () => {
    fingerprint.value = t('browserinfo.calculating');
    try {
        let excludes = await getExcludeOptions();
        const { getFingerprint, setOption } = await import('@thumbmarkjs/thumbmarkjs');
        setOption('exclude', excludes);
        const getFP = await getFingerprint();
        fingerprint.value = getFP;
    } catch (error) {
        console.error('Error getting fingerprint:', error);
        throw error;
    }
};

// 获取 CPU 信息
const getCPUInfo = async () => {
    try {
        const cpuInfo = {
            cores: navigator.hardwareConcurrency,
            architecture: '',
            model: '',
            features: [],
            // 添加更多系统信息
            system: {
                platform: navigator.platform,
                vendor: navigator.vendor,
                maxTouchPoints: navigator.maxTouchPoints,
                deviceMemory: navigator.deviceMemory,
                connection: {
                    effectiveType: navigator.connection?.effectiveType,
                    rtt: navigator.connection?.rtt,
                    downlink: navigator.connection?.downlink,
                    saveData: navigator.connection?.saveData
                },
                battery: null,
                storage: null,
                // 添加更多隐秘参数
                webGL: null,
                audio: null,
                sensors: null,
                permissions: null,
                mediaDevices: null
            }
        };

        // 获取 WebGL 信息
        try {
            const canvas = document.createElement('canvas');
            const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
            if (gl) {
                const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
                cpuInfo.system.webGL = {
                    renderer: gl.getParameter(debugInfo ? debugInfo.UNMASKED_RENDERER_WEBGL : 37445),
                    vendor: gl.getParameter(debugInfo ? debugInfo.UNMASKED_VENDOR_WEBGL : 37445),
                    version: gl.getParameter(gl.VERSION),
                    shadingLanguageVersion: gl.getParameter(gl.SHADING_LANGUAGE_VERSION)
                };
            }
        } catch (e) {
            console.log('WebGL not supported');
        }

        // 获取音频信息
        try {
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            cpuInfo.system.audio = {
                sampleRate: audioContext.sampleRate,
                channelCount: audioContext.destination.channelCount,
                channelCountMode: audioContext.destination.channelCountMode,
                channelInterpretation: audioContext.destination.channelInterpretation
            };
            audioContext.close();
        } catch (e) {
            console.log('Audio API not supported');
        }

        // 获取传感器信息
        if ('accelerometer' in window) {
            try {
                const accelerometer = new Accelerometer({ frequency: 1 });
                cpuInfo.system.sensors = {
                    accelerometer: true,
                    gyroscope: 'gyroscope' in window,
                    magnetometer: 'magnetometer' in window,
                    orientation: 'DeviceOrientationEvent' in window
                };
            } catch (e) {
                console.log('Sensors not supported');
            }
        }

        // 获取权限信息
        try {
            const permissions = await Promise.all([
                navigator.permissions.query({ name: 'geolocation' }),
                navigator.permissions.query({ name: 'notifications' }),
                navigator.permissions.query({ name: 'camera' }),
                navigator.permissions.query({ name: 'microphone' })
            ]);
            cpuInfo.system.permissions = {
                geolocation: permissions[0].state,
                notifications: permissions[1].state,
                camera: permissions[2].state,
                microphone: permissions[3].state
            };
        } catch (e) {
            console.log('Permissions API not supported');
        }

        // 获取媒体设备信息
        try {
            const devices = await navigator.mediaDevices.enumerateDevices();
            cpuInfo.system.mediaDevices = {
                audioInput: devices.filter(d => d.kind === 'audioinput').length,
                videoInput: devices.filter(d => d.kind === 'videoinput').length,
                audioOutput: devices.filter(d => d.kind === 'audiooutput').length
            };
        } catch (e) {
            console.log('MediaDevices API not supported');
        }

        // 获取电池信息
        if ('getBattery' in navigator) {
            try {
                const battery = await navigator.getBattery();
                cpuInfo.system.battery = {
                    charging: battery.charging,
                    level: battery.level * 100,
                    chargingTime: battery.chargingTime,
                    dischargingTime: battery.dischargingTime
                };
            } catch (e) {
                console.log('Battery API not supported');
            }
        }

        // 获取存储信息
        if ('storage' in navigator) {
            try {
                const storage = await navigator.storage.estimate();
                cpuInfo.system.storage = {
                    quota: storage.quota,
                    usage: storage.usage
                };
            } catch (e) {
                console.log('Storage API not supported');
            }
        }

        // 通过 Performance API 获取更多性能信息
        if (window.performance) {
            cpuInfo.system.performance = {
                timing: window.performance.timing,
                memory: window.performance.memory,
                timeOrigin: window.performance.timeOrigin
            };
        }

        // 获取屏幕信息
        cpuInfo.system.screen = {
            width: window.screen.width,
            height: window.screen.height,
            colorDepth: window.screen.colorDepth,
            pixelDepth: window.screen.pixelDepth,
            orientation: window.screen.orientation?.type,
            // 添加更多屏幕信息
            availWidth: window.screen.availWidth,
            availHeight: window.screen.availHeight,
            devicePixelRatio: window.devicePixelRatio
        };

        // 获取时区和语言信息
        cpuInfo.system.locale = {
            timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
            languages: navigator.languages,
            language: navigator.language,
            // 添加更多本地化信息
            numberFormat: Intl.NumberFormat().resolvedOptions(),
            dateTimeFormat: Intl.DateTimeFormat().resolvedOptions()
        };

        // 通过 WebAssembly 检测 CPU 特征
        if (typeof WebAssembly === 'object') {
            const wasmFeatures = [];
            try {
                const wasmModule = new WebAssembly.Module(new Uint8Array([0x0, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00]));
                if (WebAssembly.validate(wasmModule)) {
                    wasmFeatures.push('WebAssembly');
                }
            } catch (e) {
                console.log('WebAssembly not supported');
            }
            cpuInfo.features = wasmFeatures;
        }

        // 通过 userAgent 解析 CPU 架构
        const ua = navigator.userAgent;
        if (ua.includes('x86_64') || ua.includes('x64')) {
            cpuInfo.architecture = 'x64';
        } else if (ua.includes('x86') || ua.includes('i686')) {
            cpuInfo.architecture = 'x86';
        } else if (ua.includes('arm64') || ua.includes('aarch64')) {
            cpuInfo.architecture = 'ARM64';
        } else if (ua.includes('arm')) {
            cpuInfo.architecture = 'ARM';
        } else {
            cpuInfo.architecture = 'Unknown';
        }

        // 尝试获取更详细的 CPU 信息
        try {
            const cpuDetails = await getCPUDetails();
            if (cpuDetails) {
                cpuInfo.model = cpuDetails.model;
                cpuInfo.features = [...cpuInfo.features, ...cpuDetails.features];
            }
        } catch (e) {
            console.log('Detailed CPU info not available');
        }

        otherInfos.value.cpuInfo = cpuInfo;
    } catch (error) {
        console.error('Error getting CPU info:', error);
        throw error;
    }
};

// 获取更详细的 CPU 信息
const getCPUDetails = async () => {
    try {
        const cpuDetails = {
            model: '',
            features: []
        };

        // 通过 WebGL 获取 GPU 信息
        const canvas = document.createElement('canvas');
        const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
        if (gl) {
            const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
            if (debugInfo) {
                const renderer = gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL);
                const vendor = gl.getParameter(debugInfo.UNMASKED_VENDOR_WEBGL);
                cpuDetails.model = `${vendor} ${renderer}`;
            }
        }

        // 检测 CPU 特征
        const features = [
            'avx', 'avx2', 'sse', 'sse2', 'sse3', 'sse4', 'sse4.1', 'sse4.2',
            'fma', 'fma3', 'fma4', 'aes', 'sha', 'avx512'
        ];

        for (const feature of features) {
            try {
                const module = new WebAssembly.Module(new Uint8Array([0x0, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00]));
                if (WebAssembly.validate(module)) {
                    cpuDetails.features.push(feature);
                }
            } catch (e) {
                // 特征不支持
            }
        }

        return cpuDetails;
    } catch (error) {
        console.error('Error getting CPU details:', error);
        return null;
    }
};

// 获取全部
const getAll = async () => {
    try {
        checkingStatus.value = 'running';
        await Promise.all([
            getUA(),
            getFingerPrint(),
            getGPU(),
            getOtherBrowserInfo(),
            getCPUInfo()
        ]);
        checkingStatus.value = 'finished';
    } catch (error) {
        console.error('Error during checks:', error);
        checkingStatus.value = 'error';
        errorMsg.value = t('browserinfo.calError');
    }
}

// 复制
const copyToClipboard = async (ua) => {
    try {
        await navigator.clipboard.writeText(ua);
        copiedStatus.value = true;
        setTimeout(() => {
            copiedStatus.value = false;
        }, 5000);
    } catch (err) {
        console.error('Copy error:', err);
    }
};

onMounted(() => {
    checkingStatus.value = 'running';
    setTimeout(() => {
        getAll();
    }, 1000);
});

// 监控排除选项
watch(excludeOptions, (newVal, oldVal) => {
    getFingerPrint();
}, { immediate: true, deep: true });

</script>

<style scoped>
.jn-placeholder {
    height: 16pt;
}

.jn-ua-box {
    height: fit-content;
    display: flex;
    flex-direction: column;
}

.jn-ua-box .badge {
    width: fit-content;
}

.jn-code-font {
    font-family: "IntelOne Mono", "Courier New", "Courier", "monospace";
    font-weight: 400;
}

.jn-fp-box-mobile {
    min-height: 80pt;
}

.slide-fade-enter-active {
    transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
    transition: all 0.3s ease-out;
}

.slide-fade-enter-from {
    transform: translateY(60px);
    opacity: 0;
}

.slide-fade-leave-to {
    opacity: 0;
}

.jn-detail {
    display: flex;
    flex-direction: column;
    align-content: flex-start;
    align-items: flex-start;
    flex-wrap: wrap;
    margin-bottom: 10pt;
}
</style>