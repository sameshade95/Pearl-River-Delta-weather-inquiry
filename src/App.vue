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
      <div>
        <img :src="iconUrl" alt="weather" />
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
import { ref, watch } from "vue";

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

const iconUrl = ref("");

function getWeather() {
  fetch(
      `https://nd4bjaqnaq.re.qweatherapi.com/v7/weather/now?location=${cityId.value}&key=b1a2ead2e2a74b108e7a18eae7d08dd8`
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

        iconUrl.value = `https://cdn.qweather.com/weather/64/${now.icon}.png`;
      });
}

/* 关键：监听 cityId 的变化 */
watch(cityId, () => {
  getWeather();
});

/* 页面首次加载 */
getWeather();

</script>
