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
    </div>
    <div class="bg-gray-900 text-gray-400 w-full p-4 text-semibold">
      <h1 class="text-2xl">BTC fees</h1>
    </div>
  </div>
</template>

<script>
import AnimatedNumber from "animated-number-vue";
export default {
  components: {
    AnimatedNumber,
  },
  data() {
    return {
      fees: 0,
    };
  },
  mounted() {
    const ws = new WebSocket("ws://localhost:3001");

    ws.onmessage = (event) => {
      if (event.data.startsWith("{")) {
        const data = JSON.parse(event.data);
        if (data.fastestFees) {
          this.fees = data.fastestFees;
        }
      }
    };
  },
  methods: {
    formatFees(value) {
      return `${value.toFixed(2)}`;
    },
  },
};
</script>

<style>
</style>
