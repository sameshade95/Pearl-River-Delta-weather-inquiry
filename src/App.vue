<template>
  <div id="content">
    <!-- 切换城市 -->
    <div id="location">
      <select v-model="cityId">
        <option value="101010100">北京</option>
        <option value="101020100">上海</option>
        <option value="101030100">天津</option>
        <option value="101040100">重庆</option>
      </select>

    </div>

    <!-- 天气状况 -->
    <div id="weather">
      <div style="font-size: 50px;"> {{ weatherEmoji }}
      </div>
      <div>
        <div>{{ weather.text }}</div>
        <div>{{ weather.temp }} ℃</div>
      </div>
    </div>

    <!-- 实况气象数据 -->
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
import { ref, watch, computed } from "vue"; // 增加 computed

const cityId = ref("101010100");

const weather = ref({
  // ...保持原样
  icon: "100"
});

// 建立 Icon 代码与 Emoji 的映射表 (根据和风天气代码)
const emojiMap = {
  "100": "☀️", // 晴
  "101": "☁️", // 多云
  "102": "⛅", // 少云
  "103": "🌤️", // 晴间多云
  "104": "☁️", // 阴
  "300": "🌦️", // 阵雨
  "305": "🌧️", // 小雨
  "306": "🌧️", // 中雨
  "307": "🌧️", // 大雨
  "400": "🌨️", // 小雪
  "150": "🌙", // 晴（夜）
  // 可根据需要继续添加...
};

// 计算属性：匹配 Emoji，若无匹配则显示默认云朵
const weatherEmoji = computed(() => {
  return emojiMap[weather.value.icon] || "☁️";
});

function getWeather() {
  fetch(`https://nd4bjaqnaq.re.qweatherapi.com/v7/weather/now?location=${cityId.value}&key=b1a2ead2e2a74b108e7a18eae7d08dd8`)
      .then(res => res.json())
      .then(data => {
        if (data.code !== "200") return;
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
          icon: now.icon // 确保保存了 icon 编码
        };

      });
}

/* 关键：监听 cityId 的变化 */
watch(cityId, () => {
  getWeather();
});

/* 页面首次加载 */
getWeather();

</script>
