<template>
  <div class="traffic-light">
    <h2>Trafic Light App</h2>
    <Timer/>
    <Point v-for="value in signalsInformation" :key="value.number" :colorNumber="value.number"/>
  </div>
</template>

<script>
import Point from "@/components/Point";
import Timer from "@/components/Timer";
import { signalsInformation } from "@/constants";

export default {
  name: "TrafficLight",
  data() {
    return {
      signalsInformation
    };
  },
  components: { Point, Timer }
};
</script>

<style>
.traffic-light {
  display: felx;
  flex-direction: column;
  text-align: center;
}
</style>
