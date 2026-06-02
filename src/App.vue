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
      加载中…
    </div>

    <div v-else-if="error && !weather.temp" id="error">
      {{ error }}
      <button @click="getWeather">重试</button>
    </div>

    <template v-else>
      <div id="weather">
        <div style="font-size: 64px; line-height: 1;">
          {{ weatherEmoji }}
        </div>
        <div>
          <div>{{ weather.text }}</div>
          <div>{{ weather.temp }} ℃</div>
        </div>
      </div>

      <div id="now">
        <table>
          <tr>
            <td>{{ weather.feelsLike }} ℃</td>
            <td>{{ weather.humidity }} %</td>
            <td>{{ weather.precip }} mm</td>
          </tr>
          <tr>
            <td>体感温度</td>
            <td>相对湿度</td>
            <td>降水量</td>
          </tr>
          <tr>
            <td>{{ weather.pressure }} hPa</td>
            <td>{{ weather.vis }} km</td>
            <td>{{ weather.windScale }} 级</td>
          </tr>
          <tr>
            <td>气压</td>
            <td>能见度</td>
            <td>风力</td>
          </tr>
        </table>
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
