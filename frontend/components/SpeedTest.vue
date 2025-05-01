<template>
  <!-- Speed Test -->
  <div class="speed-test-section mb-4">
    <div class="jn-title2">
      <h2 id="SpeedTest" :class="{ 'mobile-h2': isMobile }">🚀 {{ t('speedtest.Title') }}</h2>
    </div>
    <div class="text-secondary">
      <p>{{ t('speedtest.Note') }}</p>
    </div>
    <div class="row">
      <div class="col-12 mb-3">
        <div class="card jn-card keyboard-shortcut-card" :class="{ 'dark-mode dark-mode-border': isDarkMode }">
          <div class="card-body">
            <!-- 控制面板 -->
            <div class="row justify-content-end mt-3 mb-4" :data-bs-theme="isDarkMode ? 'dark' : ''">
              <div class="input-group" :class="[isMobile ? 'w-100' : 'w-50']">
                <span class="input-group-text animated-icon">
                  <i class="bi bi-cloud-download"></i>
                </span>
                <select aria-label="Download Bytes" class="form-select" :class="{ 'jn-ip-font': isMobile }"
                  id="downloadBytes"
                  :disabled="state.speedTest.status === 'running' || state.speedTest.status === 'paused'"
                  v-model="state.config.package.download.bytes">
                  <option v-for="size in [100e6, 50e6, 15e6, 10e6, 1e6]" :key="size" :value="size">{{ size / 1e6 }} MB
                  </option>
                </select>
                <span class="input-group-text animated-icon">
                  <i class="bi bi-cloud-upload"></i>
                </span>
                <select aria-label="Upload Bytes" class="form-select" :class="{ 'jn-ip-font': isMobile }"
                  id="uploadBytes"
                  :disabled="state.speedTest.status === 'running' || state.speedTest.status === 'paused'"
                  v-model="state.config.package.upload.bytes">
                  <option v-for="size in [100e6, 50e6, 15e6, 10e6, 1e6]" :key="size" :value="size">{{ size / 1e6 }} MB
                  </option>
                </select>
                <button @click="speedTestController" class="btn pulse-button"
                  :class="[isDarkMode ? 'jn-startbtn-dark' : 'btn-light jn-startbtn']"
                  aria-label="Start/Pause Speed Test"
                  v-tooltip="{ title: t('Tooltips.SpeedTestButton'), placement: 'top' }">
                  <span v-if="state.speedTest.status === 'running'">
                    <i class="bi bi-pause-fill"></i>
                  </span>
                  <span v-else-if="state.speedTest.status === 'finished' || state.speedTest.status === 'error'">
                    <i class="bi bi-arrow-clockwise rotate-icon"></i>
                  </span>
                  <span v-else><i class="bi bi-caret-right-fill"></i></span>
                </button>
              </div>
            </div>

            <!-- 连接信息 -->
            <Transition name="fade-slide">
              <div class="d-flex align-items-center align-content-center justify-content-end pb-2 connection-info"
                :data-bs-theme="isDarkMode ? 'dark' : ''"
                v-if="state.speedTest.status !== 'idle' && state.connection.colo">
                <div class="connection-item">
                  <i class="bi bi-person-arms-up bounce-in"></i>
                  {{ state.connection.country }}
                  <span v-if="state.connection.country"
                    :class="'jn-fl fi fi-' + state.connection.loc.toLowerCase()"></span>
                </div>
                <div class="mx-2 connection-arrow">
                  <i class="bi bi-arrow-left-right pulse"></i>
                </div>
                <div class="connection-item">
                  <i class="bi bi-globe spin-slow"></i>
                  {{ state.connection.colo }},&nbsp;
                  {{ state.connection.coloCountry }} <span v-if="state.connection.coloCountry"
                    :class="'jn-fl fi fi-' + state.connection.coloCountryCode.toLowerCase()"></span>
                </div>
              </div>
            </Transition>

            <!-- 进度条 -->
            <div class="progress progress-container"
              :class="{ 'jn-opacity-0': state.speedTest.status == 'idle', 'jn-progress-dark': isDarkMode }">
              <div class="progress-bar progress-bar-striped jn-progress progress-animated"
                :class="[state.speedTest.status === 'finished' ? 'bg-success' : 'bg-info progress-bar-animated']"
                role="progressbar" style="width: 0%;" aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"
                id="speedtest-progress" aria-label="Progress Bar">
              </div>
            </div>

            <!-- 测试结果数值 -->
            <div class="row metrics-container">
              <div :class="['text-center metric-item', isMobile ? 'col-6' : 'col-3']">
                <p class="speedtest-h5 jn-con-title">{{ t('speedtest.Download') }}</p>
                <p id="download-speed" class="speedtest-h5 metric-value"
                  :class="updateSpeedTestColor(state.speedTest.status)">
                  <span class="jn-speedtest-number counter-animation">{{ state.speedTest.downloadSpeed }}</span>
                  <span v-if="state.speedTest.status !== 'idle'" class="metric-unit">Mb/s</span>
                </p>
              </div>
              <div :class="['text-center metric-item', isMobile ? 'col-6' : 'col-3']">
                <p class="speedtest-h5 jn-con-title">{{ t('speedtest.Upload') }}</p>
                <p id="upload-speed" class="speedtest-h5 metric-value"
                  :class="updateSpeedTestColor(state.speedTest.status)">
                  <span class="jn-speedtest-number counter-animation">{{ state.speedTest.uploadSpeed }}</span>
                  <span v-if="state.speedTest.status !== 'idle'" class="metric-unit">Mb/s</span>
                </p>
              </div>
              <div :class="['text-center metric-item', isMobile ? 'col-6' : 'col-3']">
                <p class="speedtest-h5 jn-con-title">{{ t('speedtest.Latency') }}</p>
                <p id="latency" class="speedtest-h5 metric-value" :class="updateSpeedTestColor(state.speedTest.status)">
                  <span class="jn-speedtest-number counter-animation">{{ state.speedTest.latency }}</span>
                  <span v-if="state.speedTest.status !== 'idle'" class="metric-unit">ms</span>
                </p>
              </div>
              <div :class="['text-center metric-item', isMobile ? 'col-6' : 'col-3']">
                <p class="speedtest-h5 jn-con-title">{{ t('speedtest.Jitter') }}</p>
                <p id="jitter" class="speedtest-h5 metric-value" :class="updateSpeedTestColor(state.speedTest.status)">
                  <span class="jn-speedtest-number counter-animation">{{ state.speedTest.jitter }}</span>
                  <span v-if="state.speedTest.status !== 'idle'" class="metric-unit">ms</span>
                </p>
              </div>
            </div>

            <div id="result"></div>

            <!-- 集成图表 -->
            <div class="unified-chart-container mb-4 scale-in" v-show="state.speedTest.status !== 'idle'">
              <canvas ref="unifiedChart" class="unified-chart"></canvas>
            </div>

            <!-- 测试结果 -->
            <div class="row alert alert-success m-1 p-2 result-container"
              :class="{ 'scale-in': state.speedTest.status === 'finished' }" :data-bs-theme="isDarkMode ? 'dark' : ''"
              v-if="state.speedTest.status === 'finished' && state.speedTest.hasScores">
              <p id="score" class="speedtest-p"><i class="bi bi-calendar2-check"></i>&nbsp;
                <span v-if="state.connection.colo">
                  {{ t('speedtest.connectionFrom') }}
                  {{ state.connection.ip }} ( {{ state.connection.country }} )
                  {{ t('speedtest.connectionTo') }}
                  {{ state.connection.colo }}
                  ( {{ state.connection.coloCity }}
                  , {{ state.connection.coloCountry }} )
                  {{ t('speedtest.connectionEnd') }}
                </span>
                {{ t('speedtest.score') }}
                {{ t('speedtest.videoStreaming') }}
                <span
                  :class="state.speedTest.streamingScore >= 50 ? 'text-success' : state.speedTest.streamingScore >= 10 ? 'jn-text-warning' : 'text-danger'">
                  {{ t('speedtest.quality.' + state.speedTest.streamingQuality) }}
                </span>
                {{ t('speedtest.gaming') }}
                <span
                  :class="state.speedTest.gamingScore >= 50 ? 'text-success' : state.speedTest.gamingScore >= 10 ? 'jn-text-warning' : 'text-danger'">
                  {{ t('speedtest.quality.' + state.speedTest.gamingQuality) }}
                </span>
                {{ t('speedtest.rtc') }}
                <span
                  :class="state.speedTest.rtcScore >= 50 ? 'text-success' : state.speedTest.rtcScore >= 10 ? 'jn-text-warning' : 'text-danger'">
                  {{ t('speedtest.quality.' + state.speedTest.rtcQuality) }}
                </span>
                {{ t('speedtest.resultNote') }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, computed, onMounted, markRaw, onUnmounted, ref } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';
import { trackEvent } from '@/utils/use-analytics';
import { isValidIP } from '@/utils/valid-ip.js';
import getCountryName from '@/utils/country-name.js';
import SpeedTestEngine from '@cloudflare/speedtest';
import Chart from 'chart.js/auto';

const { t } = useI18n();
const store = useMainStore();

// 计算属性
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);
const lang = computed(() => store.lang);
const isSignedIn = computed(() => store.isSignedIn);

// 图表引用
const unifiedChart = ref(null);
let chartInstance = null;

// 状态管理
const state = reactive({
  speedTest: {
    id: "speedTest",
    downloadSpeed: "-",
    uploadSpeed: "-",
    latency: "-",
    jitter: "-",
    streamingScore: "-",
    streamingQuality: "-",
    gamingScore: "-",
    gamingQuality: "-",
    rtcScore: "-",
    rtcQuality: "-",
    status: 'idle',
    hasScores: false
  },
  connection: {
    ip: "",
    colo: "",
    loc: "",
    country: "",
    coloCountry: "",
    coloCountryCode: "",
    coloCity: ""
  },
  config: {
    package: {
      download: { bytes: 50e6, count: 4 },
      upload: { bytes: 15e6, count: 4 },
      latency: { count: 30 }
    }
  },
  chartData: {
    downloadSpeeds: [],
    uploadSpeeds: [],
    latencies: [],
    jitters: []
  }
});

// 连接数据处理
const connectionMethods = {
  async getIPFromSpeedTest() {
    try {
      const response = await fetch("https://speed.cloudflare.com/cdn-cgi/trace");
      const data = await response.text();
      const lines = data.split("\n");

      const ip = lines.find(line => line.startsWith("ip="))?.split("=")[1];
      const colo = lines.find(line => line.startsWith("colo="))?.split("=")[1];
      const loc = lines.find(line => line.startsWith("loc="))?.split("=")[1];

      if (!isValidIP(ip)) {
        throw new Error("Invalid IP from SpeedTest Server");
      }

      // 动态导入 getColoCountry
      const { default: getColoCountry } = await import('@/utils/speedtest-colos.js');

      return {
        ip,
        colo,
        loc,
        country: getCountryName(loc, lang.value) || '',
        coloCountryCode: getColoCountry(colo).country || '',
        coloCity: getColoCountry(colo).city || '',
        coloCountry: getCountryName(getColoCountry(colo).country, lang.value) || ''
      };
    } catch (error) {
      console.error("Error fetching IP from SpeedTest Server:", error);
      return null;
    }
  }
};

// 图表方法
const chartMethods = {
  initChart() {
    if (chartInstance) {
      chartInstance.destroy();
    }

    const ctx = unifiedChart.value.getContext('2d');

    // 设置图表的渐变色
    const downloadGradient = ctx.createLinearGradient(0, 0, 0, 400);
    downloadGradient.addColorStop(0, 'rgba(54, 162, 235, 0.8)');
    downloadGradient.addColorStop(1, 'rgba(54, 162, 235, 0.1)');

    const uploadGradient = ctx.createLinearGradient(0, 0, 0, 400);
    uploadGradient.addColorStop(0, 'rgba(75, 192, 192, 0.8)');
    uploadGradient.addColorStop(1, 'rgba(75, 192, 192, 0.1)');

    const latencyGradient = ctx.createLinearGradient(0, 0, 0, 400);
    latencyGradient.addColorStop(0, 'rgba(255, 159, 64, 0.8)');
    latencyGradient.addColorStop(1, 'rgba(255, 159, 64, 0.1)');

    const jitterGradient = ctx.createLinearGradient(0, 0, 0, 400);
    jitterGradient.addColorStop(0, 'rgba(153, 102, 255, 0.8)');
    jitterGradient.addColorStop(1, 'rgba(153, 102, 255, 0.1)');

    chartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels: [],
        datasets: [
          {
            label: t('speedtest.Download') + ' (Mb/s)',
            data: [],
            borderColor: 'rgb(54, 162, 235)',
            backgroundColor: downloadGradient,
            borderWidth: 2,
            fill: true,
            tension: 0.4,
            pointRadius: 3,
            pointHoverRadius: 5
          },
          {
            label: t('speedtest.Upload') + ' (Mb/s)',
            data: [],
            borderColor: 'rgb(75, 192, 192)',
            backgroundColor: uploadGradient,
            borderWidth: 2,
            fill: true,
            tension: 0.4,
            pointRadius: 3,
            pointHoverRadius: 5
          },
          {
            label: t('speedtest.Latency') + ' (ms)',
            data: [],
            borderColor: 'rgb(255, 159, 64)',
            backgroundColor: latencyGradient,
            borderWidth: 2,
            fill: true,
            tension: 0.4,
            pointRadius: 3,
            pointHoverRadius: 5,
            yAxisID: 'y1'
          },
          {
            label: t('speedtest.Jitter') + ' (ms)',
            data: [],
            borderColor: 'rgb(153, 102, 255)',
            backgroundColor: jitterGradient,
            borderWidth: 2,
            fill: true,
            tension: 0.4,
            pointRadius: 3,
            pointHoverRadius: 5,
            yAxisID: 'y1'
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        animation: {
          duration: 500,
          easing: 'easeOutQuart'
        },
        interaction: {
          mode: 'index',
          intersect: false
        },
        scales: {
          x: {
            display: true,
            title: {
              display: true,
              text: t('speedtest.Time')
            },
            grid: {
              display: false
            }
          },
          y: {
            display: true,
            position: 'left',
            title: {
              display: true,
              text: 'Mb/s'
            },
            beginAtZero: true,
            grid: {
              color: isDarkMode.value ? 'rgba(255, 255, 255, 0.1)' : 'rgba(0, 0, 0, 0.1)'
            }
          },
          y1: {
            display: true,
            position: 'right',
            title: {
              display: true,
              text: 'ms'
            },
            beginAtZero: true,
            grid: {
              display: false
            }
          }
        },
        plugins: {
          legend: {
            position: 'top',
            labels: {
              usePointStyle: true,
              boxWidth: 10
            }
          },
          tooltip: {
            backgroundColor: isDarkMode.value ? 'rgba(0, 0, 0, 0.8)' : 'rgba(255, 255, 255, 0.8)',
            titleColor: isDarkMode.value ? '#fff' : '#000',
            bodyColor: isDarkMode.value ? '#fff' : '#000',
            borderColor: isDarkMode.value ? 'rgba(255, 255, 255, 0.2)' : 'rgba(0, 0, 0, 0.2)',
            borderWidth: 1,
            cornerRadius: 8,
            displayColors: true,
            animation: {
              duration: 150
            }
          }
        }
      }
    });

    return chartInstance;
  },

  updateChart(downloadSpeed, uploadSpeed, latency, jitter) {
    if (!chartInstance) return;

    // 添加时间标签
    const now = new Date();
    const timeLabel = `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`;

    // 限制数据点数量，保持图表流畅
    const maxDataPoints = 20;

    if (chartInstance.data.labels.length >= maxDataPoints) {
      chartInstance.data.labels.shift();
      chartInstance.data.datasets.forEach(dataset => dataset.data.shift());
    }

    chartInstance.data.labels.push(timeLabel);

    // 更新各项数据
    chartInstance.data.datasets[0].data.push(downloadSpeed);
    chartInstance.data.datasets[1].data.push(uploadSpeed);
    chartInstance.data.datasets[2].data.push(latency);
    chartInstance.data.datasets[3].data.push(jitter);

    // 应用动画更新图表
    chartInstance.update('active');
  },

  resetChart() {
    if (chartInstance) {
      chartInstance.data.labels = [];
      chartInstance.data.datasets.forEach(dataset => {
        dataset.data = [];
      });
      chartInstance.update();
    }
  }
};

// 测试引擎方法
let testEngine;
const engineMethods = {
  reset() {
    state.speedTest.hasScores = false;
    Object.assign(state.speedTest, {
      downloadSpeed: 0,
      uploadSpeed: 0,
      latency: 0,
      jitter: 0,
      streamingScore: "-",
      gamingScore: "-",
      rtcScore: "-"
    });

    // 清理之前的引擎实例的事件监听
    if (testEngine) {
      testEngine = null;
    }

    return markRaw(new SpeedTestEngine({
      autoStart: false,
      measurements: [
        { type: 'latency', numPackets: state.config.package.latency.count },
        { type: 'download', bytes: state.config.package.download.bytes, count: state.config.package.download.count },
        { type: 'upload', bytes: state.config.package.upload.bytes, count: state.config.package.upload.count }
      ]
    }));
  },

  updateResults(results) {
    const summary = results.getSummary();
    Object.assign(state.speedTest, {
      downloadSpeed: parseFloat((summary.download / 1000000).toFixed(2)),
      uploadSpeed: parseFloat((summary.upload / 1000000).toFixed(2)),
      latency: parseFloat(summary.latency.toFixed(2)),
      jitter: parseFloat(summary.jitter.toFixed(2))
    });
  },

  updateProgress() {
    const rawData = testEngine.results.raw;
    const progressPerStage = 100 / 3;
    let progress = 0;

    if (rawData.download?.started) {
      progress += rawData.download.finished ? progressPerStage : progressPerStage / 2;
    }
    if (rawData.upload?.started) {
      progress += rawData.upload.finished ? progressPerStage : progressPerStage / 2;
    }
    if (rawData.latency?.started) {
      progress += rawData.latency.finished ? progressPerStage : progressPerStage / 2;
    }

    progress = Math.min(progress, 100);
    this.updateProgressBar(progress);
  },

  updateProgressBar(progress) {
    const progressBar = document.getElementById('speedtest-progress');
    if (progressBar) {
      progressBar.style.width = `${progress}%`;
      progressBar.setAttribute('aria-valuenow', progress);
    }
  },

  updateSpeedInRealTime() {
    // 如果测试已结束，不再更新
    if (state.speedTest.status === 'finished' || state.speedTest.status === 'error') {
      return;
    }

    try {
      const rawData = testEngine?.results?.raw;
      if (!rawData) return;

      // 处理延迟数据和计算抖动
      if (rawData.latency?.started) {
        if (rawData.latency?.results?.timings?.length > 0) {
          const latencyTimings = rawData.latency.results.timings;
          state.speedTest.latency = parseFloat(latencyTimings[latencyTimings.length - 1].ping.toFixed(2));

          if (latencyTimings.length >= 2) {
            const differences = [];
            for (let i = 1; i < latencyTimings.length; i++) {
              differences.push(Math.abs(latencyTimings[i].ping - latencyTimings[i - 1].ping));
            }

            const mean = differences.reduce((a, b) => a + b, 0) / differences.length;
            const variance = differences.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / differences.length;
            state.speedTest.jitter = parseFloat(Math.sqrt(variance).toFixed(2));
          }
        }
      }

      // 处理下载速度
      if (rawData.download?.started) {
        if (rawData.download.current?.timings?.length > 0) {
          const timings = rawData.download.current.timings;
          state.speedTest.downloadSpeed = parseFloat((timings[timings.length - 1].bps / 1000000).toFixed(2));
        } else if (rawData.download?.results) {
          const downloadKeys = Object.keys(rawData.download.results);
          if (downloadKeys.length > 0) {
            const lastDownloadKey = downloadKeys[downloadKeys.length - 1];
            const downloadTimings = rawData.download.results[lastDownloadKey].timings;
            if (downloadTimings?.length > 0) {
              const latestTiming = downloadTimings[downloadTimings.length - 1];
              state.speedTest.downloadSpeed = parseFloat((latestTiming.bps / 1000000).toFixed(2));
            }
          }
        } else {
          state.speedTest.downloadSpeed = 0;
        }
      }

      // 处理上传速度
      if (rawData.upload?.started) {
        if (rawData.upload.current?.timings?.length > 0) {
          const timings = rawData.upload.current.timings;
          state.speedTest.uploadSpeed = parseFloat((timings[timings.length - 1].bps / 1000000).toFixed(2));
        } else if (rawData.upload?.results) {
          const uploadKeys = Object.keys(rawData.upload.results);
          if (uploadKeys.length > 0) {
            const lastUploadKey = uploadKeys[uploadKeys.length - 1];
            const uploadTimings = rawData.upload.results[lastUploadKey].timings;
            if (uploadTimings?.length > 0) {
              const latestTiming = uploadTimings[uploadTimings.length - 1];
              state.speedTest.uploadSpeed = parseFloat((latestTiming.bps / 1000000).toFixed(2));
            }
          }
        } else {
          state.speedTest.uploadSpeed = 0;
        }
      }

      // 更新图表
      chartMethods.updateChart(
        state.speedTest.downloadSpeed,
        state.speedTest.uploadSpeed,
        state.speedTest.latency,
        state.speedTest.jitter
      );

    } catch (error) {
      console.error('Error in updateSpeedInRealTime:', error);
    }
  },

  updateLatency(rawData) {
    if (!rawData.latency?.results?.timings?.length) return;
    const latencyTimings = rawData.latency.results.timings;
    const latestLatency = latencyTimings[latencyTimings.length - 1].ping;
    const newLatency = parseFloat(latestLatency.toFixed(2));
    if (newLatency < state.speedTest.latency || state.speedTest.latency === 0) {
      state.speedTest.latency = newLatency;
    }
  }
};

// 成就处理
const achievementHandler = {
  checkAndUpdate() {
    if (state.speedTest.status !== "finished") return;

    const achievements = this.getQualifiedAchievements();
    this.triggerAchievementsWithDelay(achievements);
  },

  getQualifiedAchievements() {
    const { downloadSpeed, uploadSpeed } = state.speedTest;
    const achievements = [];

    if (downloadSpeed >= 100) achievements.push('BarelyEnough');
    if (downloadSpeed >= 500) achievements.push('RapidPace');
    if (downloadSpeed >= 1000) achievements.push('TorrentFlow');
    if (uploadSpeed >= 50) achievements.push('SteadyGoing');
    if (uploadSpeed >= 200) achievements.push('TooFastTooSimple');
    if (uploadSpeed >= 1000) achievements.push('SwiftAscent');

    return achievements.filter(achievement =>
      !store.userAchievements[achievement].achieved);
  },

  triggerAchievementsWithDelay(achievements, delay = 2000) {
    if (!achievements.length) return;

    const achievement = achievements.shift();
    store.setTriggerUpdateAchievements(achievement);

    if (achievements.length) {
      setTimeout(() => this.triggerAchievementsWithDelay(achievements, delay), delay);
    }
  }
};

// 将方法直接暴露给模板使用
const updateSpeedTestColor = (status) => {
  const colorMap = {
    idle: 'text-secondary',
    running: 'text-info pulse-text',
    paused: 'text-info',
    finished: 'text-success fade-in',
    error: 'text-danger'
  };
  return colorMap[status] || '';
};

// 测试控制方法
const setupTestEngine = async () => {
  if (!state.connection.ip) {
    const connectionData = await connectionMethods.getIPFromSpeedTest();
    if (connectionData) {
      Object.assign(state.connection, connectionData);
    }
  }

  testEngine.onRunningChange = () => {
    state.speedTest.status = "running";
  };

  testEngine.onResultsChange = () => {
    engineMethods.updateProgress();
    engineMethods.updateSpeedInRealTime();
  };

  testEngine.onFinish = results => {
    // 先更新状态，防止后续更新
    state.speedTest.status = "finished";

    // 清理事件监听
    testEngine.onRunningChange = null;
    testEngine.onResultsChange = null;
    testEngine.onError = null;

    // 最后一次更新结果
    engineMethods.updateResults(results);

    const scores = results.getScores();
    if (scores?.streaming) {
      state.speedTest.hasScores = true;
      state.speedTest.streamingScore = scores.streaming.points;
      state.speedTest.gamingScore = scores.gaming.points;
      state.speedTest.rtcScore = scores.rtc.points;
      // 根据 Score 分数来判断 Quality 质量
      state.speedTest.streamingQuality = scores.streaming.points >= 50 ? 'Good' : scores.streaming.points >= 10 ? 'Medium' : 'Bad';
      state.speedTest.gamingQuality = scores.gaming.points >= 50 ? 'Good' : scores.gaming.points >= 10 ? 'Medium' : 'Bad';
      state.speedTest.rtcQuality = scores.rtc.points >= 50 ? 'Good' : scores.rtc.points >= 10 ? 'Medium' : 'Bad';
    }

    // 清理引擎实例
    testEngine = null;

    if (isSignedIn.value) {
      achievementHandler.checkAndUpdate();
    }
  };

  testEngine.onError = (e) => {
    if (typeof e === 'string' && !e.includes("ICE")) {
      state.speedTest.status = "error";
    }
    console.error('Speed Test Error: ', e);
  };
};

const speedTestController = async () => {
  try {
    if (state.speedTest.status === 'running') {
      testEngine.pause();
      state.speedTest.status = "paused";
      return;
    }

    if (state.speedTest.status === 'paused') {
      testEngine.play();
      return;
    }

    // 重置图表数据
    chartMethods.resetChart();

    Object.assign(state.speedTest, {
      downloadSpeed: 0,
      uploadSpeed: 0,
      latency: 0,
      jitter: 0,
      streamingScore: "-",
      gamingScore: "-",
      rtcScore: "-",
      hasScores: false
    });

    testEngine = engineMethods.reset();
    if (!testEngine) {
      console.error('Failed to initialize test engine');
      return;
    }

    await setupTestEngine();
    trackEvent('Section', 'StartClick', 'SpeedTest');
    testEngine.play();
  } catch (error) {
    console.error('Error in speedTestController:', error);
    state.speedTest.status = "error";
  }
};

// 生命周期
onMounted(() => {
  store.setMountingStatus('speedtest', true);
  chartMethods.initChart();
});

// 清理
onUnmounted(() => {
  if (testEngine) {
    testEngine = null;
  }
  if (chartInstance) {
    chartInstance.destroy();
    chartInstance = null;
  }
});

// 暴露方法
defineExpose({
  speedTestController
});
</script>

<style scoped>
/* 基础按钮样式 */
.jn-startbtn {
  background-color: rgb(248, 249, 250);
  border-color: rgb(222, 226, 230);
  transition: all 0.3s ease;
}

.jn-startbtn-dark {
  background-color: rgb(20, 22, 24);
  border-color: rgb(73, 80, 87);
  transition: all 0.3s ease;
}

.jn-startbtn-dark:hover {
  color: var(--bs-btn-hover-color);
  background-color: rgba(0, 0, 0, 0.33);
  transform: scale(1.05);
}

.jn-startbtn:hover {
  color: var(--bs-btn-hover-color);
  background-color: var(--bs-btn-hover-bg);
  border-color: var(--bs-btn-hover-border-color);
  transform: scale(1.05);
}

/* 警告文本颜色 */
.jn-text-warning {
  --bs-text-opacity: 1;
  color: #c67c14;
}

/* 过渡动画 */
.fade-slide-enter-active {
  transition: all 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

.fade-slide-leave-active {
  transition: all 0.5s cubic-bezier(0.55, 0.085, 0.68, 0.53);
}

.fade-slide-enter-from,
.fade-slide-leave-to {
  transform: translateY(20px);
  opacity: 0;
}

/* 图表容器 */
.unified-chart-container {
  position: relative;
  height: 300px;
  width: 100%;
  margin: 30px 0;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.unified-chart {
  width: 100%;
  height: 100%;
}

@media (max-width: 768px) {
  .unified-chart-container {
    height: 250px;
  }
}

/* 进度条容器 */
.progress-container {
  height: 20px;
  margin: 4pt 0 20pt 0;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

/* 进度条动画 */
.progress-animated {
  transition: width 0.5s ease-in-out;
  background-size: 30px 30px;
  background-image: linear-gradient(45deg,
      rgba(255, 255, 255, 0.15) 25%,
      transparent 25%,
      transparent 50%,
      rgba(255, 255, 255, 0.15) 50%,
      rgba(255, 255, 255, 0.15) 75%,
      transparent 75%,
      transparent);
  animation: progress-bar-stripes 1s linear infinite;
}

@keyframes progress-bar-stripes {
  from {
    background-position: 30px 0;
  }

  to {
    background-position: 0 0;
  }
}

/* 指标容器 */
.metrics-container {
  margin-bottom: 20px;
  display: flex;
  flex-wrap: wrap;
}

.metric-item {
  padding: 10px;
  transition: all 0.3s ease;
}

.metric-item:hover {
  transform: translateY(-5px);
}

.metric-value {
  font-size: 1.8rem;
  font-weight: bold;
  margin: 0;
  transition: all 0.3s ease;
}

.metric-unit {
  font-size: 1rem;
  opacity: 0.7;
}

/* 连接信息样式 */
.connection-info {
  padding: 8px 12px;
  border-radius: 8px;
  background-color: rgba(0, 0, 0, 0.03);
  margin-bottom: 15px;
  transition: all 0.3s ease;
}

.connection-item {
  display: flex;
  align-items: center;
  gap: 5px;
}

/* 结果容器 */
.result-container {
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 动画效果 */
.pulse-button {
  position: relative;
}

.pulse-button::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border-radius: inherit;
  animation: pulse 1.5s infinite;
  z-index: -1;
  opacity: 0;
}

@keyframes pulse {
  0% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(0, 123, 255, 0.7);
    opacity: 0.7;
  }

  70% {
    transform: scale(1);
    box-shadow: 0 0 0 10px rgba(0, 123, 255, 0);
    opacity: 0;
  }

  100% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(0, 123, 255, 0);
    opacity: 0;
  }
}

.pulse-text {
  animation: pulse-text 1.5s infinite;
}

@keyframes pulse-text {
  0% {
    opacity: 1;
  }

  50% {
    opacity: 0.7;
  }

  100% {
    opacity: 1;
  }
}

.rotate-icon {
  display: inline-block;
  transition: transform 0.3s ease;
}

.rotate-icon:hover {
  transform: rotate(180deg);
}

.animated-icon {
  transition: all 0.3s ease;
}

.animated-icon:hover {
  transform: scale(1.2);
}

.bounce-in {
  animation: bounce-in 0.5s;
}

@keyframes bounce-in {
  0% {
    transform: scale(0);
  }

  50% {
    transform: scale(1.2);
  }

  100% {
    transform: scale(1);
  }
}

.spin-slow {
  animation: spin 8s linear infinite;
  display: inline-block;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

.connection-arrow i {
  animation: pulse 2s infinite;
}

.scale-in {
  animation: scale-in 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards;
}

@keyframes scale-in {
  0% {
    transform: scale(0.9);
    opacity: 0;
  }

  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.counter-animation {
  display: inline-block;
  position: relative;
  transition: all 0.3s ease;
}

.counter-animation::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 100%;
  height: 2px;
  background: currentColor;
  transform: scaleX(0);
  transition: transform 0.3s ease;
}

.counter-animation:hover::after {
  transform: scaleX(1);
}

.fade-in {
  animation: fadeIn 0.5s ease-in forwards;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}
</style>