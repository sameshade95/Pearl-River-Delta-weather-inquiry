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
  </div>
</template>

<script setup>
import { ref, watch, computed } from "vue";

const cityId = ref("101010100");

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

// 天气代码转 Emoji 逻辑
const getWeatherEmoji = (code) => {
  const c = parseInt(code);

  // 1. 特殊处理夜间图标 (和风天气夜间代码通常以 15x, 35x, 45x 等开头)
  const nightIcons = {
    "150": "🌙", // 晴（夜）
    "151": "☁️", // 多云（夜）
    "152": "☁️", // 少云（夜）
    "153": "☁️", // 晴间多云（夜）
    "350": "🌧️", // 阵雨（夜）
    "351": "🌧️", // 强阵雨（夜）
    "456": "🌨️", // 阵雪（夜）
    "457": "🌨️", // 强阵雪（夜）
  };

  if (nightIcons[code]) {
    return nightIcons[code];
  }

  // 2. 常规白天或通用图标逻辑
  if (c === 100) return "☀️";            // 晴
  if (c >= 101 && c <= 199) return "☁️";  // 云
  if (c >= 300 && c <= 399) return "🌧️";  // 雨
  if (c >= 400 && c <= 499) return "🌨️";  // 雪
  if (c >= 500 && c <= 599) return "🌫️";  // 雾/霾
  if (c >= 200 && c <= 299) return "💨";  // 风

  return "🌡️";
};

// 计算属性：根据当前 icon 实时计算 Emoji
const weatherEmoji = computed(() => {
  return getWeatherEmoji(weather.value.icon);
});

function getWeather() {
  fetch(
      `https://这里填API host/v7/weather/now?location=${cityId.value}&key=这里填API key`
  )
      .then(res => res.json())
      .then(data => {
        if (data.code !== "200") {
          console.error("API 返回错误：", data);
          return;
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
      });
}

watch(cityId, () => {
  getWeather();
});

getWeather();
</script>

