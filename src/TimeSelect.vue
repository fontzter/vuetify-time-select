<template>
  <v-input hide-details class="time-select" :color="color">
    <template v-if="label" v-slot:label>
      <div class="caption">{{ label }}</div>
    </template>
    <v-menu offset-y>
      <template v-slot:activator="{ on }">
        <v-btn outlined class="time-hours" :color="color" v-on="on">
          <span>{{ value ? hour : "H" }}</span>
          <v-icon right>mdi-menu-down</v-icon>
        </v-btn>
      </template>
      <v-list dense>
        <v-list-item
          v-for="hr in 12"
          :key="hr"
          class="pr-0 pl-2"
          @click="hour = hr === 1 ? 12 : hr - 1"
          >{{ hr === 1 ? 12 : hr - 1 }}
        </v-list-item>
      </v-list>
    </v-menu>
    <div class="">:</div>
    <v-menu
      v-model="minMenu"
      offset-y
      max-height="80vh"
      :close-on-content-click="false"
    >
      <template v-slot:activator="{ on: menu }">
        <v-btn outlined class="time-minutes" :color="color" v-on="menu">
          <span>{{ value ? minute : "M" }}</span>
          <v-icon right>mdi-menu-down</v-icon>
        </v-btn>
      </template>
      <v-list dense>
        <v-list-item
          v-for="min in minuteList"
          :key="min"
          class="pr-0 pl-2"
          :class="getMinuteClass(min)"
          @click="onMinuteClick(min)"
        >
          <v-icon v-if="min.includes('-')">mdi-dots-horizontal</v-icon>
          <span v-else>{{ min }}</span>
        </v-list-item>
      </v-list>
    </v-menu>
    <v-btn-toggle v-model="isAm" class="toggle-am-pm" :color="color">
      <v-btn :value="true" class="btn-am">am</v-btn>
      <v-btn :value="false" class="btn-pm">pm</v-btn>
    </v-btn-toggle>
  </v-input>
</template>

<script>
export default {
  name: "TimeSelect",
  props: {
    color: {
      type: String,
      default: null
    },
    label: {
      type: String,
      default: "Time"
    },
    minuteGroups: {
      type: Array,
      default: () => [0, 15, 30, 45]
    },
    value: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      expandedMinutes: null,
      minMenu: false
    };
  },
  computed: {
    hour24() {
      if (!this.$props.value) return 12;
      return Number(this.$props.value.split(":")[0]);
    },
    hour: {
      get() {
        return this.hour24 % 12 === 0 ? 12 : this.hour24 % 12;
      },
      set(value) {
        if (value) {
          const hr = `${(value === 12 ? 0 : value) +
            (this.isAm ? 0 : 12)}`.padStart(2, "0");
          this.$emit("input", `${hr}:${this.minute}`);
        }
      }
    },
    expandedStart() {
      return this.expandedMinutes
        ? parseInt(this.expandedMinutes.split("-")[1])
        : -1;
    },
    expandedEnd() {
      return this.expandedMinutes
        ? parseInt(this.expandedMinutes.split("-")[2])
        : -1;
    },
    minuteList() {
      return Array.from(Array(60).keys())
        .map(i => {
          if (
            this.minuteGroups.includes(i) ||
            (i >= this.expandedStart && i <= this.expandedEnd)
          ) {
            return `${i}`.padStart(2, "0");
          } else if (this.minuteGroups.includes(i - 1)) {
            const end = this.minuteGroups.find(a => a > i) || 59;
            return `ellipsis-${i}-${end}`;
          } else {
            return false;
          }
        })
        .filter(i => !!i);
    },
    minute: {
      get() {
        if (!this.$props.value) return "00";
        return this.$props.value.split(":")[1].replace(/\D+/g, "");
      },
      set(value) {
        if (value) {
          this.$emit(
            "input",
            `${("" + this.hour24).padStart(2, "0")}:${value}`
          );
        }
      }
    },
    isAm: {
      get() {
        return this.hour24 < 12;
      },
      set(toAm) {
        if (toAm === this.isAm) return; // avoid if not changing
        const hr = `${this.hour24 + (toAm ? -12 : 12)}`.padStart(2, "0");
        this.$emit("input", `${hr}:${this.minute}`);
      }
    }
  },
  methods: {
    onMinuteClick(min) {
      const isEllipsis = min.includes("-");
      this.expandedMinutes = isEllipsis ? min : null;
      if (!isEllipsis) {
        this.minMenu = false;
        this.minute = min;
      }
    },
    getMinuteClass(min) {
      return this.minuteGroups.includes(parseInt(min))
        ? "font-weight-bold"
        : "";
    }
  }
};
</script>

<style>
.time-select {
  padding-top: 12px;
  margin-top: 4px;
}
.time-select label {
  margin-top: -50px;
  margin-right: -28px;
}
.time-hours,
.time-minutes {
  width: 40px;
  min-width: 40px !important;
  height: 32px !important;
}
.time-hours .v-btn__content i.v-icon,
.time-minutes .v-btn__content i.v-icon {
  margin-left: -4px;
}

.time-select .toggle-am-pm .v-btn.v-size--default {
  width: 36px;
  min-width: 36px !important;
  height: 32px !important;
  padding-left: 0;
  padding-right: 0;
}
</style>
