<template>
  <div>
    <div class="loading" v-if="loading">
      <v-progress-circular
        indeterminate
        color="blue"
        size="72"
      ></v-progress-circular>
    </div>

    <v-container style="padding: 0; margin: 0">
      <v-row>
        <v-col cols="12" :height="this.height" class="tradeCol" sm="6">
          <trading-vue
            :data="this.$data"
            :width="this.width"
            :height="this.height"
            ref="tradingVue"
          ></trading-vue>
        </v-col>
        <v-col cols="12" sm="2" style="padding-left: 20px; padding-top: 1rem">
          <v-row cols="12">
            <v-col sm="12" xs="5">
              <div class="options pa-sm-1">
                <v-autocomplete
                  v-model="stocks"
                  :items="items"
                  item-text="name"
                  item-value="id"
                  label="Stocks"
                  hint="Select your stock"
                  return-object
                  outlined
                ></v-autocomplete>
              </div>
            </v-col>

            <v-col sm="12" xs="5" style="padding-left: 20px">
              <div class="options">
                <v-select
                  style="margin-top: 0"
                  v-model="selectedOptions"
                  :items="options"
                  :menu-props="{ maxHeight: '400' }"
                  label="Select"
                  multiple
                  hint="Pick your models"
                  persistent-hint
                ></v-select>
              </div>
            </v-col>
          </v-row>
        </v-col>
        <v-col cols="12" sm="4" style="margin-top: 20px">
          <div class="data" v-if="apiData.length > 0">
            <v-card style="width: 100%">
              <v-row>
                <v-col cols="6">
                  <h4 style="width: 200px">Current price:</h4>
                  <h4 style="width: 200px">Current RSI:</h4>
                  <h4 style="width: 200px">7-day price:</h4>
                  <h4 style="width: 200px">Change 7-day:</h4>
                  <h4 style="width: 200px">Last years price:</h4>
                  <h4 style="width: 200px">Change last year:</h4>
                  <h4 style="width: 200px">What to do:</h4>
                </v-col>
                <v-col cols="6">
                  <h4 style="width: 150px">
                    {{ apiData[apiData.length - 1][4].toFixed(2) }}
                  </h4>
                  <h4 style="width: 150px">
                    {{ rsiData[rsiData.length - 1] }}
                  </h4>
                  <h4 style="width: 150px">{{ weekChange[4].toFixed(2) }}</h4>
                  <h4
                    style="width: 150px"
                    :style="{
                      color: getColor(
                        (apiData[apiData.length - 1][4] / weekChange[4]) * 100
                      ),
                    }"
                  >
                    {{
                      (
                        (apiData[apiData.length - 1][4] / weekChange[4]) *
                        100
                      ).toFixed(2)
                    }}%
                  </h4>
                  <h4 style="width: 150px">{{ yearChange[4].toFixed(2) }}</h4>
                  <h4
                    style="width: 150px"
                    :style="{
                      color: getColor(
                        (apiData[apiData.length - 1][4] / yearChange[4]) * 100
                      ),
                    }"
                  >
                    {{
                      (
                        (apiData[apiData.length - 1][4] / yearChange[4]) *
                        100
                      ).toFixed(2)
                    }}%
                  </h4>
                  <h4 style="width: 150px">
                    {{ prediction }}
                  </h4>
                </v-col>
              </v-row>
            </v-card>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>
<script>
import { TradingVue } from "trading-vue-js";
import { EMA, RSI } from "technicalindicators";

var axios = require("axios");
const timezoneData = require("../assets/tz.json");
const jsonStocks = require("../assets/stockJson.json");

let data = [];

export default {
  name: "app",
  components: { TradingVue },

  data() {
    return {
      selectedOptions: "",
      options: ["RSI", "3-Line Strike"],
      items: jsonStocks,
      tz: timezoneData[26],
      loading: false,
      rsiData: [],
      stocks: "",
      timezoneData: timezoneData,
      apiData: [],
      weekChange: "",
      yearChange: "",
      prediction: "",
      titleText: "Market Manipulator2k",
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight,

      ohlcv: data,
      onchart: [
        {
          name: "EMA 9",
          type: "EMA",
          data: [],
          settings: { color: "white" },
        },
        {
          name: "EMA 20",
          type: "EMA",
          data: [],
          settings: { color: "yellow" },
        },
        {
          name: "EMA 50",
          type: "EMA",
          data: [],
          settings: { color: "purple" },
        },
        {
          name: "EMA 200",
          type: "EMA",
          data: [],
          settings: { color: "orange" },
        },
      ],
      offchart: [{ name: "RSI", type: "RSI", data: [] }],
    };
  },
  methods: {
    getDimensions() {
      this.width = document.querySelector(".tradeCol").offsetWidth;
      this.height = document.querySelector(".tradeCol").offsetHeight;
    },
    getColor(v) {
      if (v < 100) {
        return "red";
      } else if (v > 100) {
        return "green";
      } else {
        return "white";
      }
    },
    async fetchData(id) {
      return await axios.request({
        method: "GET",
        url:
          "https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=" +
          id +
          "&interval=1d&apikey=FUOIZZHV806NY77H&outputsize=full",
      });
    },
    async insertData(id) {
      this.apiData = [];
      const res = await this.fetchData(id);
      // console.log(res.data)
      for (let [timestamp, value] of Object.entries(
        res.data["Time Series (Daily)"]
      )) {
        // console.log(value)
        let date = new Date(timestamp);
        // console.log(value)
        // console.log(value["1. open"])
        this.apiData.push([
          date.getTime(), //+ (parseInt(this.tz.value) + 4) * 60 * 60 * 1000,
          parseFloat(value["1. open"]),
          parseFloat(value["2. high"]),
          parseFloat(value["3. low"]),
          parseFloat(value["4. close"]),
        ]);
      }

      this.apiData.reverse();
      this.ohlcv = this.apiData;

      const closeData = [];
      this.apiData.forEach((d) => closeData.push(d[4]));

      this.onchart[0].data = [];
      const ema9Data = new EMA.calculate({ period: 9, values: closeData });
      this.apiData.forEach((d, i) => {
        const emad = i > 9 ? ema9Data[i - 9] : undefined;
        this.onchart[0].data.push([d[0], emad]);
      });

      this.onchart[1].data = [];
      const ema20Data = new EMA.calculate({ period: 20, values: closeData });
      this.apiData.forEach((d, i) => {
        const emad = i > 20 ? ema20Data[i - 20] : undefined;
        this.onchart[1].data.push([d[0], emad]);
      });

      this.onchart[2].data = [];
      const ema50Data = new EMA.calculate({ period: 50, values: closeData });
      this.apiData.forEach((d, i) => {
        const emad = i > 50 ? ema50Data[i - 50] : undefined;
        this.onchart[2].data.push([d[0], emad]);
      });

      this.onchart[3].data = [];
      const ema200Data = new EMA.calculate({ period: 200, values: closeData });
      this.apiData.forEach((d, i) => {
        const emad = i > 200 ? ema200Data[i - 200] : undefined;
        this.onchart[3].data.push([d[0], emad]);
      });

      var inputRSI = {
        values: closeData,
        period: 14,
      };
      this.offchart[0].data = [];
      this.rsiData = RSI.calculate(inputRSI);
      this.apiData.forEach((d, i) => {
        const emad = i > 14 ? this.rsiData[i - 14] : undefined;
        this.offchart[0].data.push([d[0], emad]);
      });

      if (
        this.rsiData[this.rsiData.length - 2] === undefined ||
        this.rsiData[this.rsiData.length - 1] === undefined
      ) {
        this.prediction = "N/A";
      } else if (
        parseFloat(this.rsiData[this.rsiData.length - 2]) < 30 &&
        parseFloat(this.rsiData[this.rsiData.length - 1]) >= 30
      ) {
        this.prediction = "Buy";
      } else if (
        parseFloat(this.rsiData[this.rsiData.length - 2]) > 70 &&
        parseFloat(this.rsiData[this.rsiData.length - 1]) <= 70
      ) {
        this.prediction = "Sell";
      } else {
        this.prediction = "Hold";
      }

      this.$refs.tradingVue.resetChart();
    },

    candleStickPred() {
      for (let i = 1; i <= 3; i++) {
        let v =
          this.apiData[this.apiData.length - i][4] -
          this.apiData[this.apiData.length - i][1];

        if (parseFloat(v) <= 0) {
          return false;
        }
      }
      return true;
    },
    async updateData(id) {
      this.loading = true;
      await this.insertData(id);
      await this.getStats();
      this.loading = false;
    },
    async getStats() {
      this.weekChange = this.apiData[this.apiData.length - 4];
      this.yearChange = this.apiData[this.apiData.length - (5 * 52 - 7)];
    },
  },
  // created() {},
  mounted() {
    this.getDimensions();
    this.tz = JSON.parse(localStorage.getItem("timezone"));
    this.selectedOptions = JSON.parse(localStorage.getItem("options"));
    window.addEventListener("resize", this.getDimensions, { passive: true });
  },
  unmounted() {
    window.removeEventListener("resize", this.getDimensions, { passive: true });
  },
  watch: {
    selectedOptions() {
      console.log(this.selectedOptions);
      localStorage.setItem("options", JSON.stringify(this.selectedOptions));
    },
    stocks() {
      if (this.stocks == null) return;
      this.updateData(this.stocks.id);
    },
  },
};
</script>
<style>
.noPadding {
  padding: 0 !important;
  /* padding-bottom: 1px !important; */
  margin: 0 !important;
}
.options {
  z-index: 10;
  left: 82vw;
  top: 2vh;
  max-width: 11rem;
}

.loading {
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 11;
}
</style>
