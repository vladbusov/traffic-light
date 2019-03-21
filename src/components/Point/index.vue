<template>
  <div
    class="point-circle"
    :class="{
      'red': colorForPoint === 'red',
      'yellow': colorForPoint === 'yellow',
      'green': colorForPoint === 'green',
      'active': checkActivePoint,
      'white': blinkingOn
    }"
  ></div>
</template>

<script>
import { signalsInformation, blinkingTime } from "@/constants";
import { eventEmitter } from "@/main";

export default {
  name: "Point",
  props: {
    colorNumber: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      routeNumber: this.$router.currentRoute.params["number"],
      blinkingTimerInstance: null,
      blinkingOn: false
    };
  },
  methods: {
    isExistColorNumber(arr, number) {
      let result = arr.find(value => +value === +number);
      return result;
    },
    toggleSignal(arraySignals, currentSignalNumber) {
      const currentPoint = arraySignals.find(
          value => +value.number === +currentSignalNumber
        ),
        // определение следующего сигнала
        nextPoint =
          +currentSignalNumber < +arraySignals.length
            ? ++currentSignalNumber
            : 1;

      return new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve(nextPoint);
        }, currentPoint.lifeTime * 1000);
      });
    },
    activateBlinkingPoint() {
      this.blinkingTimerInstance = setTimeout(() => {
        this.blinkingOn = !this.blinkingOn;
        this.activateBlinkingPoint();
      }, 100);
    }
  },
  watch: {
    $route(toR, fromR) {
      let colorNumbers = signalsInformation.map(value => value.number),
        isExistNumber = this.isExistColorNumber(
          colorNumbers,
          toR.params["number"]
        );
      if (isExistNumber) {
        const colorObject = signalsInformation.find(
          value => +value.number === +toR.params["number"]
        );

        this.routeNumber = toR.params["number"];

        // +toR.params["number"] === +this.colorNumber &&
        eventEmitter.$emit("startShowSignal", colorObject);

        this.toggleSignal(signalsInformation, this.routeNumber).then(
          nextSignalNumber => {
            // при изменении сигнала очищаем данные о мигании текущей точки
            this.blinkingTimerInstance &&
              (clearTimeout(this.blinkingTimerInstance),
              (this.blinkingOn = false),
              (this.blinkingTimerInstance = null));

            //переход к следующему состоянию
            this.$router.push(`/${nextSignalNumber}`);
          }
        );
      } else {
        this.$router.push("/1");
      }
    }
  },
  computed: {
    checkActivePoint() {
      return +this.routeNumber === +this.colorNumber ? true : false;
    },
    colorForPoint() {
      let colorForPoint = "";
      if (this.colorNumber === 1) {
        colorForPoint = "red";
      } else if (this.colorNumber === 2) {
        colorForPoint = "yellow";
      } else if (this.colorNumber === 3) {
        colorForPoint = "green";
      }
      return colorForPoint;
    }
  },
  mounted() {
    const colorNumbers = signalsInformation.map(value => value.number),
      isExistNumberRoute = this.isExistColorNumber(
        colorNumbers,
        this.$router.currentRoute.params["number"]
      );

    if (isExistNumberRoute) {
      const colorObject = signalsInformation.find(
        value => +value.number === +this.$router.currentRoute.params["number"]
      );

      +this.routeNumber === +this.colorNumber &&
        eventEmitter.$emit("startShowSignal", colorObject);

      const toggleSignal = this.toggleSignal(
        signalsInformation,
        this.$router.currentRoute.params["number"]
      );
      toggleSignal.then(nextSignalNumber => {
        // при изменении сигнала очищаем данные о мигании текущей точки
        this.blinkingTimerInstance &&
          (clearTimeout(this.blinkingTimerInstance),
          (this.blinkingOn = false),
          (this.blinkingTimerInstance = null));

        // переход к следующему состоянию
        this.$router.push(`/${nextSignalNumber}`);
      });
    } else {
      this.$router.push("/1");
    }
  },
  created() {
    eventEmitter.$on("startBlinking", activePointNumber => {
      if (+activePointNumber === +this.colorNumber) {
        this.activateBlinkingPoint();
      }
    });
  }
};
</script>

<style>
.point-circle {
  width: 150px;
  height: 150px;
  margin: auto;
  border-radius: 50%;
  border: 1px solid black;
  margin-top: 20px;
}

.red {
  background-color: red;
  opacity: 0.5;
}

.yellow {
  background-color: yellow;
  opacity: 0.5;
}

.green {
  background-color: green;
  opacity: 0.5;
}

.active {
  opacity: 1;
}

.white {
  background-color: white;
}
</style>
