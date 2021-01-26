<template>
  <div class="flex flex-col w-full min-h-screen">
    <div
      class="bg-gray-900 text-gray-400 w-full justify-center flex flex-col flex-grow h-full items-center"
    >
      <div class="text-6xl">
        <animated-number
          :value="fees"
          :formatValue="formatFees"
          :duration="500"
        />
      </div>
      <div class="text-2xl">sat/vB</div>

      <div class="w-full md:w-1/2 h-56">
        <div class="ct-chart ct-perfect-fourth h-56"></div>
      </div>

      <div>Last block mined {{ timeSinceLastBlockFormat }} ago</div>
    </div>

    <div
      class="bg-gray-900 text-gray-400 w-full p-4 text-semibold flex justify-between items-baseline"
    >
      <h1 class="text-2xl">Bitcoin fastest fee estimate</h1>
      <div class="flex items-baseline text-red-500">
        <a
          href="https://github.com/dolu89/btc-fees-front"
          target="_blank"
          class="hover:text-red-700"
          >Contribute</a
        >
        <a
          href="https://twitter.com/Dolu_Web"
          target="_blank"
          class="hover:text-red-700 ml-5"
          >@Dolu_Web</a
        >
      </div>
    </div>
  </div>
</template>

<script>
import AnimatedNumber from "animated-number-vue";
import Chartist from "chartist";
import "chartist-logaxis";
import ctPointLabels from "chartist-plugin-pointlabels";
import "chartist/dist/chartist.min.css";

export default {
  components: {
    AnimatedNumber,
  },
  data() {
    return {
      fees: 0,
      timeSinceLastBlock: 0,
    };
  },
  mounted() {
    const ws = new WebSocket(process.env.wsApiUrl);

    ws.onmessage = (event) => {
      if (event.data.startsWith("{")) {
        const data = JSON.parse(event.data);
        this.timeSinceLastBlock = data.timeSinceLastBlock;
        this.fees = data.fastestFee;
        const feeRange = data.nextBlock.feeRange.map((fee) => Math.trunc(fee));

        const chartData = {
          labels: feeRange,
          series: [
            {
              name: "feeRange",
              data: feeRange,
            },
            {
              name: "fastestFee",
              data: data.nextBlock.feeRange.map((e) => data.fastestFee),
            },
          ],
        };

        const chartOptions = {
          low: 0,
          height: "225px",
          fullWidth: true,
          axisX: {
            showLabel: false,
            showGrid: false,
          },
          axisY: {
            type: Chartist.AutoScaleAxis,
            scale: "log10",
          },
          series: {
            feeRange: {
              showArea: false,
              showPoint: true,
            },
            fastestFee: {
              showLine: true,
              showArea: true,
              showPoint: false,
            },
          },
          plugins: [
            ctPointLabels({
              labelClass: "ct-label-point",
              textAnchor: "middle",
            }),
          ],
        };
        new Chartist.Line(".ct-chart", chartData, chartOptions);
      }
    };
  },
  methods: {
    formatFees(value) {
      return `${value.toFixed(2)}`;
    },
  },
  computed: {
    timeSinceLastBlockFormat() {
      let minutes = Math.trunc(this.timeSinceLastBlock / 60);
      if (minutes === 0){
        minutes = '< 1'
      }
      const minutesString = minutes > 1 ? "minutes" : "minute";
      return `${minutes} ${minutesString}`;
    },
  },
};
</script>

<style>
.ct-label {
  color: #e2e8f0;
}

.ct-label-point {
  fill: #e2e8f0;
  color: #e2e8f0;
  font-size: 14px;
}

.ct-series-a .ct-line {
  stroke: #e53e3e;
  stroke-width: 2px;
}

.ct-series-a .ct-point {
  stroke: #c53030;
  stroke-width: 7px;
}

.ct-series-b .ct-line {
  stroke-width: 1px;
  stroke: #fbd38d;
}

.ct-series-b .ct-area {
  fill: #feebc8;
}
</style>
