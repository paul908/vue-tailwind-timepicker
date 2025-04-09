<script setup lang="ts">
import { ref } from "vue";
import ClockDial from "./components/ClockDial.vue";
import InputTimePicker from "./components/InputTimePicker.vue";

const is24h = ref<boolean>(true);
const hour = ref<number>(12);
const minute = ref<number>(30);
const open = ref<boolean>(true);
const pm = ref<boolean>(false);
const time = ref<string>('12:30');

const mode = ref<'hour' | 'minute'>('hour');
const hourMinute = ref<{hour:number, minute:number}>({
  hour: 12,
  minute: 30
});

const DEBUG = true;

function debugLog(...args: any) {
  if (DEBUG) console.log(...args);
}


const handleUpdate = (event: any) => {
  hourMinute.value.hour = event.hour;
  hourMinute.value.minute = event.minute;
};

const onUpdatePm = (value: boolean) => {
  debugLog('onUpdatePm triggered with value:', value);
  pm.value = value;
  if (value) {
    if (pm.value) {
      hour.value = hour.value > 12 ? hour.value - 12 : hour.value;
    }
  }
};

const onSwitch = () => {
  debugLog('Switch triggered');
  mode.value = mode.value === 'hour' ? 'minute' : 'hour';
}
</script>

<template>
  <InputTimePicker v-model:is24h="is24h"  v-model:time="time"/>

  <!-- <ClockDial :is24h="is24h" :pm="pm" :hour="hour" :minute="minute" :mode="mode" :open="open"
    @update:onUpdate="handleUpdate" @updatePm="onUpdatePm" @switch="onSwitch" /> -->
</template>

<style scoped></style>
