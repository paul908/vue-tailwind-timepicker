<script setup lang="ts">
import {computed, ref, watch} from "vue";
import InputTimePicker from "./components/InputTimePicker.vue";
import Switch from "./components/Switch.vue";
import TimePicker from "./components/TimePicker.vue";

/**
 * App component
 * @description A simple app that demonstrates the use of the InputTimePicker and TimePicker components.
 * @example <App />
 * @param {boolean} is24h - Whether to use 24-hour format or not.
 * @param {string} time - The time value in HH:MM format.
 */

/**
 * @param {boolean} is24h - Whether to use 24-hour format or not.
 * @param {string} time - The time value in HH:MM format.
 * @param {boolean} pm - Whether the time is in PM or AM.
 */
const is24h = ref<boolean>(true);
const pm = ref<boolean>(false);
const time = ref<string>('12:30');

/**
 * @description Debug log function
 * @param {...any} args - The arguments to log.
 * @returns {void}
 */
const DEBUG = false;

function debugLog(...args: any) {
  if (DEBUG) console.log(...args);
}

/**
 * @description computed property for pmLabel
 * @returns {string} - The label for PM or AM.
 */
const pmLabel = computed(() => {
  debugLog("TimePicker.vue pmLabel: ", pm.value ? 'PM' : 'AM');
  return pm.value ? 'PM' : 'AM';
})

/**
 * @description computed property for ampmHour
 * @returns {number} - The hour in 12-hour format.
 */
const ampmHour = computed(() => {
  let x = hourMinute.value.hour;
  if (x > 12) {
    x -= 12;
  } else if (x === 0 || x === 24) {
    x = 12;
  }
  return x
})

/**
 * @description computed property for hourMinute
 * @returns {{hour: number, minute: number}} - The hour and minute values.
 */
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

// Watchers

/**
 * @description Watcher for is24h
 * @returns {void}
 */
watch(() => is24h.value, () => {
  debugLog('index.vue is24h: ', is24h.value);
})

/**
 * @description Watcher for time
 * @returns {void}
 */
watch(() => time.value, () => {
  debugLog('InputTimePicker watch(() => time.value: ', time.value);
})

/**
 * @description Watcher for hourMinute.hour
 * @returns {void}
 */
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
    <!-- Mobile Phone Styled Container -->
    <div class="mobile-phone">
      <div class="flex justify-between items-center px-3 h-7 bg-gray-100 text-black text-xs font-medium rounded-t-[18px] border-b border-gray-300">
        <div class="text-left">
          {{ hourMinute.hour }}:{{ hourMinute.minute.toString().padStart(2, '0') }}
        </div>
        <div class="text-right space-x-1">
          <span>📶</span>
          <span>📡</span>
          <span>🔋</span>
        </div>
      </div>

      <div class="flex flex-col h-[90%] items-center justify-center overflow-y-scroll">
        <!--    InputTimePicker -->
        <InputTimePicker v-model:is24h="is24h" v-model:time="time "/>
        <div>
          <!-- Time Display 24h -->
          <div
              class=" flex flex-col items-center justify-center mx-auto m-4 p-4 font-semibold text-neutral-600">
            <!--    Switch -->
            <div class="flex items-center justify-center space-x-2 m-4 border rounded-lg p-2 w-36">
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
        </div>
        <!-- TimePicker -->
        <TimePicker v-model:is24h="is24h" v-model:time="time "/>

      </div>
      <!-- iPhone-style home indicator -->
      <div class="h-5 flex items-center justify-center">
        <div class="w-20 h-1.5 bg-gray-400 rounded-full"></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.mobile-phone {
  width: 340px;
  height: 640px;
  background: #f0f0f0;
  border: 12px solid #333;
  border-radius: 30px;
  position: relative;
  margin: 20px auto;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
}
</style>
