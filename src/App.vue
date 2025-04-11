<script setup lang="ts">
import {computed, onMounted, ref, watch} from "vue";
import InputTimePicker from "./components/InputTimePicker.vue";
import Switch from "./components/Switch.vue";
import TimePicker from "./components/TimePicker.vue";

const is24h = ref<boolean>(true);
const pm = ref<boolean>(false);
const time = ref<string>('12:30');

const DEBUG = false;

function debugLog(...args: any) {
  if (DEBUG) console.log(...args);
}

var primaryColor = '';
var secondaryColor = '';
var neutralColor = '';
var textColor = '';


onMounted(() => {
  const root = getComputedStyle(document.documentElement);
  primaryColor = root.getPropertyValue('--ui-color-primary-500').trim();
  secondaryColor = root.getPropertyValue('--ui-color-secondary-500').trim();
  neutralColor = root.getPropertyValue('--ui-color-neutral-200').trim();
  textColor = root.getPropertyValue('--ui-color-neutral-700').trim();
  debugLog("primaryColor: ", primaryColor);
})

const pmLabel = computed(() => {
  debugLog("TimePicker.vue pmLabel: ", pm.value ? 'PM' : 'AM');
  return pm.value ? 'PM' : 'AM';
})

const ampmHour = computed(() => {
  let x = hourMinute.value.hour;
  if (x > 12) {
    x -= 12;
  } else if (x === 0 || x === 24) {
    x = 12;
  }
  return x
})

// Structured format, e.g., { hour: 13, minute: 45 }
const hourMinute = computed({
  get() {
    debugLog('compute hourMinute: ', time.value);
    if (!time.value) return {hour: 0, minute: 0}
    const [h, m] = time.value.split(':')
    debugLog('index.vue time: ', time.value, ' - hour: ', h, ' - minute: ', m)
    return {
      hour: Number(h) || 0,
      minute: Number(m) || 0,
    }
  },
  set(val) {
    const {hour, minute} = val
    time.value = `${String(hour).padStart(2, '0')}:${String(minute).padStart(2, '0')}`
    debugLog('index.vue time: ', time.value, ' - hour: ', hour, ' - minute: ', minute)
  }
})

watch(() => is24h.value, () => {
  debugLog('index.vue is24h: ', is24h.value);
})

watch(() => time.value, () => {
  debugLog('InputTimePicker watch(() => time.value: ', time.value);
})

watch(() => hourMinute.value.hour, () => {
  debugLog("InputTimePicker.vue watch hourMinute.value.hour: ", hourMinute.value?.hour);
  if (hourMinute.value) {
    pm.value = hourMinute.value.hour >= 12;
    debugLog("new pm.value: ", pm.value);
  }
})

</script>

<template>
  <div class="flex flex-col items-center justify-center h-screen overflow-y-scroll">

    <!--    InputTimePicker -->
    <InputTimePicker v-model:is24h="is24h" v-model:time="time "/>

    <!-- Time Display 24h -->
    <div class="flex flex-col items-center justify-center mx-auto p-4">
      <!--    Switch -->
      <div class="flex items-center space-x-2">
        <Switch v-model="is24h" label-enabled="24h" label-disabled="AM/PM"/>
      </div>
      <div v-if="is24h">
        <p class="h-16 text-6xl text-center my-auto">{{
            hourMinute.hour
          }}:{{ hourMinute.minute.toString().padStart(2, '0') }}</p>
      </div>
      <div v-else>
        <p class="h-16 text-6xl text-center my-auto">{{
            ampmHour
          }}:{{ hourMinute.minute.toString().padStart(2, '0') }}&nbsp;{{ pmLabel }}</p>
      </div>
    </div>

    <!-- TimePicker -->
    <TimePicker v-model:is24h="is24h" v-model:time="time "/>
  </div>
</template>

<style scoped></style>
