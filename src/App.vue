<template>
  <div id="content">
    <div id="location">
      <select v-model="cityId">
        <option value="101280101">廣州</option>
        <option value="101280601">深圳</option>
        <option value="101280701">珠海</option>
        <option value="101280800">佛山</option>
        <option value="101281601">東莞</option>
        <option value="101281701">中山</option>
        <option value="101320101">香港</option>
        <option value="101330101">澳門</option>
      </select>
    </div>

    <div v-if="loading && !weather.temp" id="loading">
      <div class="loading-spinner"></div>
      <div>加载中…</div>
    </div>

    <div v-else-if="error && !weather.temp" id="error">
      <div class="error-icon">⚠️</div>
      <div class="error-message">{{ error }}</div>
      <button @click="getWeather">重试</button>
    </div>

    <template v-else>
      <div id="weather">
        <div class="weather-icon">{{ weatherEmoji }}</div>
        <div class="weather-temp">{{ weather.temp }}°C</div>
        <div class="weather-text">{{ weather.text }}</div>
      </div>

      <div id="details">
        <div class="detail-card temp">
          <div class="detail-icon">🌡️</div>
          <div class="detail-value">{{ weather.feelsLike }}°</div>
          <div class="detail-label">体感温度</div>
        </div>
        <div class="detail-card humid">
          <div class="detail-icon">💧</div>
          <div class="detail-value">{{ weather.humidity }}%</div>
          <div class="detail-label">相对湿度</div>
        </div>
        <div class="detail-card precip">
          <div class="detail-icon">🌧️</div>
          <div class="detail-value">{{ weather.precip }} mm</div>
          <div class="detail-label">降水量</div>
        </div>
        <div class="detail-card press">
          <div class="detail-icon">📊</div>
          <div class="detail-value">{{ weather.pressure }} hPa</div>
          <div class="detail-label">气压</div>
        </div>
        <div class="detail-card vis">
          <div class="detail-icon">👁️</div>
          <div class="detail-value">{{ weather.vis }} km</div>
          <div class="detail-label">能见度</div>
        </div>
        <div class="detail-card wind">
          <div class="detail-icon">💨</div>
          <div class="detail-value">{{ weather.windScale }} 级</div>
          <div class="detail-label">风力</div>
        </div>
      </div>
    </template>

    <div v-if="loading && weather.temp" id="refreshing">
      刷新中…
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed, onUnmounted } from "vue";

const cityId = ref("101280101");

const weather = ref({
  temp: "",
  feelsLike: "",
  text: "",
  humidity: "",
  precip: "",
  pressure: "",
  vis: "",
  windScale: "",
  icon: "100"
});

const loading = ref(false);
const error = ref("");

let abortController = null;

const getWeatherEmoji = (code) => {
  const c = parseInt(code);

  const nightIcons = {
    "150": "🌙",
    "151": "☁️",
    "152": "☁️",
    "153": "☁️",
    "350": "🌧️",
    "351": "🌧️",
    "456": "🌨️",
    "457": "🌨️",
  };

  if (nightIcons[code]) {
    return nightIcons[code];
  }

  if (c === 100) return "☀️";
  if (c >= 101 && c <= 199) return "☁️";
  if (c >= 300 && c <= 399) return "🌧️";
  if (c >= 400 && c <= 499) return "🌨️";
  if (c >= 500 && c <= 599) return "🌫️";
  if (c >= 200 && c <= 299) return "💨";

  return "🌡️";
};

const weatherEmoji = computed(() => {
  return getWeatherEmoji(weather.value.icon);
});

function getWeather() {
  abortController?.abort();
  abortController = new AbortController();

  loading.value = true;
  error.value = "";

  fetch(
      `https://${import.meta.env.VITE_API_HOST}/v7/weather/now?location=${cityId.value}&key=${import.meta.env.VITE_API_KEY}`,
      { signal: abortController.signal }
  )
      .then(res => res.json())
      .then(data => {
        if (data.code !== "200") {
          throw new Error(data.code === "403"
              ? "API Key 无效或未配置"
              : `请求失败 (${data.code})`);
        }

        const now = data.now;
        weather.value = {
          temp: now.temp,
          feelsLike: now.feelsLike,
          text: now.text,
          humidity: now.humidity,
          precip: now.precip,
          pressure: now.pressure,
          vis: now.vis,
          windScale: now.windScale,
          icon: now.icon
        };
        loading.value = false;
      })
      .catch(err => {
        if (err.name === "AbortError") return;
        error.value = err.message || "网络请求失败，请检查网络连接";
        loading.value = false;
      });
}

watch(cityId, () => {
  getWeather();
});

getWeather();

onUnmounted(() => {
  abortController?.abort();
});
</script>
