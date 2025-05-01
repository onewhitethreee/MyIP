<template>
    <div class="modal fade" id="mapModal" tabindex="-1" aria-labelledby="mapModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-lg modal-dialog-centered">
            <div class="modal-content" :class="{ 'dark-mode': isDarkMode }">
                <!-- Enhanced Header -->
                <div class="modal-header">
                    <h5 class="modal-title" id="mapModalLabel">
                        <i class="bi bi-geo-alt-fill me-2 text-primary"></i>
                        {{ t('map.Location') }}
                    </h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>

                <!-- Enhanced Body -->
                <div class="modal-body p-0">
                    <!-- Map Provider Selector Tabs -->
                    <div class="map-tabs">
                        <div class="provider-tabs">
                            <button v-for="provider in mapProviders" :key="provider.value"
                                :class="['provider-tab', { active: mapProvider === provider.value }]"
                                @click="mapProvider = provider.value">
                                <i :class="provider.icon"></i>
                                {{ t(provider.label) }}
                            </button>
                        </div>
                    </div>

                    <!-- Map Container with Loading Animation -->
                    <div class="map-container" :class="{ 'loading': !showMap }">
                        <!-- Google Maps -->
                        <div v-if="mapProvider === 'google' && showMap" class="map-frame">
                            <iframe :src="googleMapsUrl" width="100%" height="450" style="border:0;" allowfullscreen=""
                                loading="lazy" referrerpolicy="no-referrer-when-downgrade">
                            </iframe>
                        </div>

                        <!-- OpenStreetMap -->
                        <div v-else-if="mapProvider === 'osm' && showMap" class="map-frame">
                            <iframe :src="osmUrl" width="100%" height="450" style="border:0;" allowfullscreen=""
                                loading="lazy">
                            </iframe>
                        </div>

                        <!-- Baidu Maps -->
                        <div v-else-if="mapProvider === 'baidu' && showMap" class="map-frame">
                            <iframe :src="baiduMapsUrl" width="100%" height="450" style="border:0;" allowfullscreen=""
                                loading="lazy">
                            </iframe>
                        </div>

                        <!-- Enhanced Loading Animation -->
                        <div v-if="!showMap" class="map-placeholder">
                            <div class="loading-animation">
                                <div class="spinner"></div>
                                <p class="loading-text">{{ t('map.LoadingMap') || 'Loading map...' }}</p>
                            </div>
                        </div>
                    </div>

                    <!-- Enhanced Location Info Card -->
                    <div class="location-card">
                        <div class="location-details">
                            <div class="location-info">
                                <div class="info-item">
                                    <i class="bi bi-geo-alt-fill text-primary"></i>
                                    <span>{{ location.address }}</span>
                                </div>
                                <div class="info-item">
                                    <i class="bi bi-flag-fill text-success"></i>
                                    <span>{{ location.country }}</span>
                                </div>
                                <div class="info-item">
                                    <i class="bi bi-building text-info"></i>
                                    <span>{{ location.organization }}</span>
                                </div>
                                <div class="info-item coordinates">
                                    <i class="bi bi-geo text-secondary"></i>
                                    <span>{{ location.latitude }}, {{ location.longitude }}</span>
                                </div>
                            </div>

                            <!-- Action Buttons -->
                            <div class="action-buttons">
                                <button class="action-btn copy-btn" @click="copyCoordinates">
                                    <i class="bi bi-clipboard"></i>
                                    {{ t('map.CopyCoordinates') || 'Copy' }}
                                </button>
                                <a :href="directionsUrl" target="_blank" class="action-btn primary">
                                    <i class="bi bi-signpost-2"></i>
                                    {{ t('map.GetDirections') || 'Directions' }}
                                </a>
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
import { Modal } from 'bootstrap';

const { t } = useI18n();
const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);

const props = defineProps({
    location: {
        type: Object,
        required: true,
        default: () => ({
            latitude: 0,
            longitude: 0,
            address: '',
            country: '',
            organization: ''
        })
    }
});

const mapProvider = ref('google');
const showMap = ref(false);
let modalInstance = null;

// Map providers configuration
const mapProviders = [
    { value: 'google', label: 'map.GoogleMaps', icon: 'bi bi-google' },
    { value: 'osm', label: 'map.OpenStreetMap', icon: 'bi bi-map' },
    { value: 'baidu', label: 'map.BaiduMaps', icon: 'bi bi-globe' }
];

// Computed map URLs
const googleMapsUrl = computed(() => {
    if (!props.location.latitude || !props.location.longitude) return '';
    return `https://www.google.com/maps/embed/v1/place?key=${import.meta.env.VITE_GOOGLE_MAPS_API_KEY}&q=${props.location.latitude},${props.location.longitude}&zoom=15`;
});

const osmUrl = computed(() => {
    if (!props.location.latitude || !props.location.longitude) return '';
    const size = 0.01;
    return `https://www.openstreetmap.org/export/embed.html?bbox=${props.location.longitude - size},${props.location.latitude - size},${props.location.longitude + size},${props.location.latitude + size}&layer=mapnik&marker=${props.location.latitude},${props.location.longitude}`;
});

const baiduMapsUrl = computed(() => {
    if (!props.location.latitude || !props.location.longitude) return '';
    return `https://api.map.baidu.com/staticimage/v2?ak=${import.meta.env.VITE_BAIDU_MAPS_API_KEY}&center=${props.location.longitude},${props.location.latitude}&width=800&height=450&zoom=15&markers=${props.location.longitude},${props.location.latitude}`;
});

// Directions URL based on current provider
const directionsUrl = computed(() => {
    if (!props.location.latitude || !props.location.longitude) return '#';

    if (mapProvider.value === 'google') {
        return `https://www.google.com/maps/dir/?api=1&destination=${props.location.latitude},${props.location.longitude}`;
    } else if (mapProvider.value === 'osm') {
        return `https://www.openstreetmap.org/directions?engine=graphhopper_foot&route=;${props.location.latitude},${props.location.longitude}`;
    } else if (mapProvider.value === 'baidu') {
        return `https://api.map.baidu.com/direction?origin=&destination=${props.location.latitude},${props.location.longitude}&mode=driving&region=global&output=html`;
    }

    return '#';
});

// Handle map provider change
watch(mapProvider, () => {
    showMap.value = false;
    setTimeout(() => {
        showMap.value = true;
    }, 300);
});

// Copy coordinates to clipboard
const copyCoordinates = () => {
    const text = `${props.location.latitude}, ${props.location.longitude}`;
    navigator.clipboard.writeText(text).then(() => {
        const copyBtn = document.querySelector('.action-btn.copy-btn');
        if (copyBtn) {
            copyBtn.classList.add('copy-success');
            setTimeout(() => {
                copyBtn.classList.remove('copy-success');
            }, 2000);
        }
    });
};

onMounted(() => {
    let modalElement = null;
    try {
        modalElement = document.getElementById('mapModal');
    } catch (error) {
        console.error("Error getting modal element:", error);
    }

    if (modalElement) {
        try {
            modalInstance = new Modal(modalElement, {
                backdrop: true,
                keyboard: true
            });
        } catch (error) {
            console.error("Error creating modal instance:", error);
        }

        // Listen for modal events
        modalElement.addEventListener('shown.bs.modal', () => {
            showMap.value = true;
        });

        modalElement.addEventListener('hidden.bs.modal', () => {
            showMap.value = false;
        });
    }
});

// Expose methods to parent component
defineExpose({
    show: () => {
        if (modalInstance) {
            modalInstance.show();
        }
    },
    hide: () => {
        if (modalInstance) {
            modalInstance.hide();
        }
    }
});
</script>

<style scoped>
/* Base Styles */
.modal-content {
    border: none;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.modal-header {
    padding: 1rem 1.5rem;
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.modal-title {
    font-weight: 600;
    display: flex;
    align-items: center;
}

.modal-body {
    padding: 0;
}

/* Map Container */
.map-container {
    position: relative;
    min-height: 450px;
    background-color: #f8f9fa;
    transition: all 0.3s ease;
}

.map-container.loading {
    filter: blur(2px);
}

.map-frame {
    width: 100%;
    height: 450px;
    border: none;
}

/* Loading Animation */
.map-placeholder {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: rgba(255, 255, 255, 0.7);
    z-index: 10;
}

.loading-animation {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.spinner {
    width: 40px;
    height: 40px;
    border: 4px solid rgba(0, 0, 0, 0.1);
    border-radius: 50%;
    border-top-color: var(--bs-primary, #0d6efd);
    animation: spin 1s ease-in-out infinite;
}

.loading-text {
    margin-top: 1rem;
    font-size: 0.9rem;
    color: #666;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}

/* Map Provider Tabs */
.map-tabs {
    background-color: #f8f9fa;
    padding: 0.5rem 1rem;
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.provider-tabs {
    display: flex;
    gap: 0.5rem;
    overflow-x: auto;
    padding-bottom: 0.25rem;
}

.provider-tab {
    background: none;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 6px;
    font-size: 0.9rem;
    cursor: pointer;
    white-space: nowrap;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: #666;
    transition: all 0.2s ease;
}

.provider-tab:hover {
    background-color: rgba(0, 0, 0, 0.05);
}

.provider-tab.active {
    background-color: var(--bs-primary, #0d6efd);
    color: white;
}

/* Location Info Card */
.location-card {
    padding: 1.5rem;
    background-color: white;
}

.location-details {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
}

.location-info {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1rem;
}

.info-item {
    display: flex;
    align-items: flex-start;
    gap: 0.75rem;
    font-size: 0.95rem;
}

.info-item i {
    margin-top: 0.2rem;
    font-size: 1rem;
}

.coordinates {
    font-family: monospace;
    font-size: 0.85rem;
    color: #666;
}

/* Action Buttons */
.action-buttons {
    display: flex;
    gap: 1rem;
    margin-top: 0.5rem;
    justify-content: flex-end;
}

.action-btn {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 1rem;
    border-radius: 6px;
    font-size: 0.9rem;
    cursor: pointer;
    text-decoration: none;
    border: 1px solid rgba(0, 0, 0, 0.1);
    background-color: white;
    color: #333;
    transition: all 0.2s ease;
    position: relative;
}

.action-btn:hover {
    background-color: #f8f9fa;
}

.action-btn.primary {
    background-color: var(--bs-primary, #0d6efd);
    color: white;
    border-color: var(--bs-primary, #0d6efd);
}

.action-btn.primary:hover {
    background-color: var(--bs-primary-dark, #0b5ed7);
}

.action-btn.copy-success {
    background-color: var(--bs-success, #198754);
    color: white;
    border-color: var(--bs-success, #198754);
}

.action-btn.copy-success::after {
    content: '✓';
    position: absolute;
    right: 0.5rem;
    animation: fadeIn 0.2s ease-in-out;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateX(-10px);
    }
    to {
        opacity: 1;
        transform: translateX(0);
    }
}

/* Dark Mode Styles */
.dark-mode .modal-content {
    background-color: #222;
    color: #f8f9fa;
}

.dark-mode .modal-header {
    border-bottom-color: rgba(255, 255, 255, 0.1);
}

.dark-mode .map-tabs {
    background-color: #2c2c2c;
    border-bottom-color: rgba(255, 255, 255, 0.1);
}

.dark-mode .provider-tab {
    color: #ccc;
}

.dark-mode .provider-tab:hover {
    background-color: rgba(255, 255, 255, 0.1);
}

.dark-mode .location-card {
    background-color: #2c2c2c;
}

.dark-mode .map-placeholder {
    background-color: rgba(0, 0, 0, 0.7);
}

.dark-mode .loading-text {
    color: #ccc;
}

.dark-mode .action-btn {
    background-color: #333;
    color: #f8f9fa;
    border-color: rgba(255, 255, 255, 0.1);
}

.dark-mode .action-btn:hover {
    background-color: #444;
}

/* Responsive Design */
@media (max-width: 768px) {
    .modal-dialog {
        margin: 0.5rem;
        max-width: calc(100% - 1rem);
    }

    .location-info {
        grid-template-columns: 1fr;
    }

    .action-buttons {
        flex-direction: column;
    }

    .map-frame {
        height: 350px;
    }

    .map-container {
        min-height: 350px;
    }
}
</style>