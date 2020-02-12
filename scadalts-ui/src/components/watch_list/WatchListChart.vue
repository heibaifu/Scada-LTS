<template>
  <div class="flex-container">
    <div class="settings" v-if="renderChart">
      <img src="/ScadaBR/images/cog.png" @click="showSettings()" class="settings-btn">
      <img src="/ScadaBR/images/delete.png" @click="deleteChart()" class="settings-btn">
    </div>
    <div v-if="showModal" class="settings-modal">
      <div class="settings-modal-container">
        <div class="flex-radio">
          <div class="flex-radio">
            <label for="static-rbtn">Step Line Chart</label>
            <input
              type="radio"
              class="radio-button"
              value="stepLine"
              id="typeStep-rbtn"
              v-model="chartSettings.lineChart"
            />
          </div>
          <div class="flex-radio">
            <label for="type-rbtn">Line Chart</label>
            <input
              type="radio"
              class="radio-button"
              value="line"
              id="type-rbtn"
              v-model="chartSettings.lineChart"
            />
          </div>
        </div>
        <hr />
        <div class="flex-radio">
          <div class="flex-radio">
            <label for="live-rbtn">Live Chart</label>
            <input
              type="radio"
              class="radio-button"
              value="live"
              id="live-rbtn"
              v-model="chartSettings.chartType"
            />
          </div>
          <div class="flex-radio">
            <label for="static-rbtn">Static Chart</label>
            <input
              type="radio"
              class="radio-button"
              value="static"
              id="static-rbtn"
              v-model="chartSettings.chartType"
            />
          </div>
        </div>
        <div v-if="chartSettings.chartType === 'live'" class="flex-container">
          <div>
            <label for="live-sd">Values from last: </label>
            <input type="number" id="live-sd" v-model="chartSettings.startTime" />
            <select v-model="chartSettings.startTimeMultiplier">
              <option v-for="option in timeOptions" v-bind:value="option.value" v-bind:key="option.id">
                {{option.text}}
              </option>
            </select>
          </div>
          <div>
            <label for="live-rr">Chart performance</label>
            <select id="live-rr" v-model="chartSettings.refreshRate">
              <option v-for="option in performanceOptions" v-bind:value="option.value" v-bind:key="option.id">
                {{option.text}}
              </option>
            </select>
          </div>
        </div>
        <div v-if="chartSettings.chartType === 'static'" class="flex-container">
          <div>
            <label for="static-sd">Start Date</label>
            <datepicker format="yyyy/MM/dd" :monday-first="true" v-model="chartSettings.startDate"></datepicker>
          </div>
          <div>
            <label for="static-ed">End Date</label>
            <datepicker format="yyyy/MM/dd" :monday-first="true" v-model="chartSettings.endDate"></datepicker>
          </div>
        </div>
        <div class="flex-container">
          <div>
            <label for="chart-color">Chart color</label>
            <ColorPicker
              :width="100"
              :height="100"
              startColor="#39B54A"
              v-model="chartSettings.chartColor"
            ></ColorPicker>
          </div>
          <div>
            <label for="chart-rv">Range value</label>
            <input type="text" id="chart-rv" v-model="chartSettings.rangeValue" />
          </div>
          <div v-if="chartSettings.rangeValue">
            <label for="chart-rc">Range color</label>
            <ColorPicker
              :width="100"
              :height="100"
              startColor="#ff0000"
              v-model="chartSettings.rangeColor"
            ></ColorPicker>
          </div>
          <div v-if="chartSettings.rangeValue">
            <label for="chart-rl">Range label</label>
            <input type="text" id="chart-rl" v-model="chartSettings.rangeLabel" />
          </div>
          <div>
            <label for="chart-height">Chart heigh</label>
            <input type="number" id="chart-height" v-model="chartSettings.height" />
          </div>
        </div>
        <div class="settings">
          <button @click="cancelSettings()"  class="modal-settings-btn"><img src="/ScadaBR/images/cross.png"  class="settings-btn"> Close</button>
          <button @click="saveSettings()"  class="modal-settings-btn"><img src="/ScadaBR/images/accept.png" class="settings-btn"> Save</button>
        </div>
      </div>
    </div>
    <div v-if="renderChart">
      <line-chart
        v-bind:point-id="chartdata.pointId"
        v-bind:label="chartdata.chartLabel"
        v-bind:start-date="chartdata.startDate"
        v-bind:end-date="chartdata.endDate"
        v-bind:refresh-rate="chartdata.refreshRate"
        v-bind:color="chartdata.chartColor"
        v-bind:range-value="chartdata.rangeValue"
        v-bind:range-color="chartdata.rangeColor"
        v-bind:range-label="chartdata.rangeLabel"
        v-bind:show-scrollbar-x="chartdata.showScrollbarX"
        v-bind:show-scrollbar-y="chartdata.showScrollbarY"
        v-bind:show-legend="chartdata.showLegend"
        v-bind:height="chartdata.height"
        v-if="chartdata.lineChart == 'line'"
        ref="line_child"
      />
      <step-line-chart
        v-bind:point-id="chartdata.pointId"
        v-bind:label="chartdata.chartLabel"
        v-bind:start-date="chartdata.startDate"
        v-bind:end-date="chartdata.endDate"
        v-bind:refresh-rate="chartdata.refreshRate"
        v-bind:color="chartdata.chartColor"
        v-bind:range-value="chartdata.rangeValue"
        v-bind:range-color="chartdata.rangeColor"
        v-bind:range-label="chartdata.rangeLabel"
        v-bind:show-scrollbar-x="chartdata.showScrollbarX"
        v-bind:show-scrollbar-y="chartdata.showScrollbarY"
        v-bind:show-legend="chartdata.showLegend"
        v-bind:height="chartdata.height"
        v-bind:show-debug="chartdata.debug"
        v-if="chartdata.lineChart == 'stepLine'"
        ref="step_line_child"
      />
    </div>
  </div>
</template>
<script>
import Axios from "axios";
import StepLineChart from "../amcharts/StepLineChartComponent";
import LineChart from "../amcharts/LineChartComponent";
import Datepicker from "vuejs-datepicker";
import ColorPicker from "vue-color-picker-wheel";
export default {
  name: "WatchListChart",
  components: {
    StepLineChart,
    LineChart,
    Datepicker,
    ColorPicker
  },
  props: ["chartdata"],
  data() {
    return {
      pointId: undefined,
      chartSettings: undefined,
      points: [],
      renderChart: true,
      showModal: false,
      performanceOptions: [
        {id: 0, text: "Real Time", value: 1000},
        {id: 1, text: "High resolution", value: 2000},
        {id: 2, text: "Standard", value: 5000},
        {id: 3, text: "Faster performance", value: 10000}
      ],
      timeOptions: [
        {id: 0, text: "Hour(s)", value: "hour"},
        {id: 1, text: "Day(s)", value: "day"},
        {id: 2, text: "Weak(s)", value: "weak"},
        {id: 3, text: "Month(s)", value: "month"}
      ]
    };
  },
  mounted() {
    this.showChart();
  },
  methods: {
    getPointIds() {
      this.points = [];
      let wachList = document.getElementById("watchListTable");
      for (let i = 0; i < wachList.childElementCount; i++) {
        let point = wachList.children.item(i).id;
        if (document.getElementById(`${point}ChartCB`).checked) {
          this.points.push(point.slice(1));
        }
      }
      this.chartdata.pointId = this.points.toString();
    },
    showChart() {
      this.renderChart = true;
    },
    showSettings() {
      this.chartSettings = this.chartdata;
      this.showModal = true;
      this.renderChart = false;
    },
    cancelSettings() {
      if(this.chartdata.chartColor == "#3973b5" || this.chartdata.chartColor == "#39B54A") {
        this.chartdata.chartColor = undefined;
      }
      this.showModal = false;
      this.renderChart = true;
    },
    saveSettings() {
      
      if (this.chartSettings.chartType == "static") {
        this.chartSettings.refreshRate = undefined;
        let date = this.chartSettings.startDate;
        this.chartSettings.startDate =
          date.getFullYear() +
          "/" +
          (date.getMonth() + 1) +
          "/" +
          date.getDate();
        date = this.chartSettings.endDate;
        this.chartSettings.endDate =
          date.getFullYear() +
          "/" +
          (date.getMonth() + 1) +
          "/" +
          date.getDate();
      } else {
        this.chartSettings.endDate = undefined;
        this.chartSettings.startDate = `${this.chartdata.startTime}-${this.chartdata.startTimeMultiplier}`
      }
      if(this.chartSettings.chartColor == "#3973b5" || this.chartSettings.chartColor == "#39B54A") {
        this.chartSettings.chartColor = undefined;
      }
      if(this.chartSettings.rangeLabel == "DEBUG_CHART") {
        this.chartSettings.debug = true;
      }
      this.chartdata = this.chartSettings;
      this.showModal = false;
      this.renderChart = true;
      this.$emit("saved", this.chartdata);     
    },
    deleteChart() {
      this.$emit("deleted", this.chartdata)
    }
  }
};
</script>
<style scoped>
.flex-container {
  display: flex;
  flex-direction: column;
}
.settings {
  display: flex;
  justify-content: flex-end;
}
.flex-radio {
  display: flex;
  justify-content: space-between;
  width: 98%;
  padding: 0 1%;
}
.settings-modal {
  position: fixed;
  z-index: 999;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.4);
  transition: opacity 0.3s ease-in-out;
}
.settings-modal-container {
  width: 400px;
  margin: 0 auto;
  margin-top: 30px;
  max-height: 700px;
  padding: 20px 30px;
  background-color: #ffffff;
  border-radius: 5px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
}
.settings-modal-container > .flex-container > div {
  padding: 10px 0;
  display: flex;
  justify-content: space-between;
}
button {
  color: #333333;
  border: 1px solid #39b54a;
  padding: 2px;
  min-width: 100px;
}
.settings-btn {
  width: 16px;
  height: 16px;
}
</style>