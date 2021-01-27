<template>
  <div class="flex flex-col w-full min-h-screen">
    <div
      class="bg-gray-900 text-gray-400 w-full justify-center flex flex-col flex-grow h-full items-center"
      v-show="!isLoading"
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

      <div id="legends"></div>

      <div>Last block mined {{ timeSinceLastBlockFormat }} ago</div>
    </div>

    <div
      class="bg-gray-900 text-gray-400 w-full justify-center flex flex-col flex-grow h-full items-center"
      v-show="isLoading"
    >
      <span class="blink">Fetching data...</span>
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
import PersistentWebSocket from "pws";
import AnimatedNumber from "animated-number-vue";
import Chartist from "chartist";
import "chartist/dist/chartist.min.css";
import "chartist-plugin-legend";
import "chartist-logaxis";
import "chartist-plugin-axistitle";
import ctPointLabels from "chartist-plugin-pointlabels";

export default {
  components: {
    AnimatedNumber,
  },
  data() {
    return {
      isLoading: true,
      fees: 0,
      timeSinceLastBlock: 0,
    };
  },
  mounted() {
    const chartOptions = {
      chartPadding: {
        right: 40,
        left: 20,
      },
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
        Chartist.plugins.legend({
          legendNames: ["Next block fees range", "Optimized fee"],
          position: document.getElementById("legends"),
        }),
        Chartist.plugins.ctAxisTitle({
          axisY: {
            axisTitle: "sat / vB",
            axisClass: "ct-axis-title",
          },
        }),
      ],
    };
    let chartist = null;

    const ws = new PersistentWebSocket(process.env.wsApiUrl);

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

        this.isLoading = false;
        if (chartist) {
          chartist.update(chartData);
        } else {
          chartist = new Chartist.Line(".ct-chart", chartData, chartOptions);
        }
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
      if (minutes === 0) {
        minutes = "< 1";
      }
      const minutesString = minutes > 1 ? "minutes" : "minute";
      return `${minutes} ${minutesString}`;
    },
  },
};
</script>

<style lang="scss">
.ct-label {
  color: #e2e8f0;
}
.ct-axis-title {
  fill: #e2e8f0;
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

.blink {
  animation: blinker 1s linear infinite;
  transform: translateX(-50%) translateY(-50%);
}

@keyframes blinker {
  50% {
    opacity: 0;
  }
}

#legends {
  margin-top: -30px;
  margin-bottom: 20px;
}

.ct-legend {
  list-style: none;
  justify-content: center;
  display: flex;
  align-items: center;

  li {
    padding-left: 23px;
    margin-right: 10px;
    margin-bottom: 3px;
    position: relative;
    display: flex;
    align-items: center;
    cursor: pointer;

    &:before {
      width: 12px;
      height: 12px;
      position: absolute;
      left: 0;
      content: "";
      border: 3px solid transparent;
      border-radius: 2px;
    }
    .inactive:before {
      background: transparent;
    }

    &:nth-child(1)::before {
      background-color: #e53e3e;
    }

    &:nth-child(2)::before {
      background-color: #fbd38d;
    }
  }

  // .ct-legend-inside {
  //   position: absolute;
  //   top: 0;
  //   right: 0;
  // }
}
// g:not(.ct-grids):not(.ct-labels) g {
//   &:nth-child(1) {
//     .ct-point,
//     .ct-line {
//       stroke: #d70206;
//     }
//   }
//   &:nth-child(2) {
//     .ct-point,
//     .ct-line {
//       stroke: #f05b4f;
//     }
//   }
//   &:nth-child(3) {
//     .ct-point,
//     .ct-line {
//       stroke: #f4c63d;
//     }
//   }
//   &:nth-child(1n + 4) {
//     .ct-point,
//     .ct-line {
//       stroke: #f06292;
//     }
//   }
// }
</style>
