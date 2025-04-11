<script setup lang="ts">
import {ref, computed, onMounted, watch} from 'vue'
import Checkbox from "../src/components/Checkbox.vue";
import ClockDial from "../src/components/ClockDial.vue";
import Switch from "../src/components/Switch.vue";

const DEBUG = false;

function debugLog(...args: any) {
  if (DEBUG) console.log(...args);
}

const hourMinute = defineModel<{ hour: number, minute: number }>('hourMinute')
const is24h = defineModel<boolean>('is24h')

const safeHourMinute = computed(() => {
  const h = hourMinute.value?.hour ?? 0
  const m = hourMinute.value?.minute ?? 0
  debugLog('TimePicker.vue safeHourMinute: ', hourMinute.value ?? {hour: 0, minute: 0});
  return {hour: h, minute: m}
})

const safeHour = computed(() => {
  const h = hourMinute.value?.hour ?? 0
  debugLog('TimePicker.vue safeHour: ', h);
  return h;
})

const paddedHour = computed(() => {
  const h = hourMinute.value?.hour ?? 0
  debugLog('TimePicker.vue paddedHour: ', String(h).padStart(2, '0'));
  return `${String(h).padStart(2, '0')}`
})

const safeMinute = computed(() => {
  const m = hourMinute.value?.minute ?? 0
  debugLog('TimePicker.vue safeMinute: ', m);
  return m;
})

const paddedMinute = computed(() => {
  const m = hourMinute.value?.minute ?? 0
  debugLog('TimePicker.vue paddedMinute: ', String(m).padStart(2, '0'));
  return `${String(m).padStart(2, '0')}`
})

const safeIs24h = computed(() => {
  debugLog('TimePicker.vue safeIs24h: ', is24h.value ?? true);
  return is24h.value ?? true
})

const ampmHour = computed(() => {
  if (hourMinute.value) {
    let x = hourMinute.value.hour;
    if (x > 12) {
      x -= 12;
    } else if (x === 0) {
      x = 12;
    }
    return x
  } else return 0;
})

const isOpen = ref(false)
const selecting = ref<'hour' | 'minute'>('hour')
const localHour = ref(0)
const localMinute = ref(0)

const pm = ref(false)

pm.value = safeHourMinute.value.hour >= 12

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
  debugLog('TimePicker.vue pmLabel: ', pm.value ? 'PM' : 'AM');
  return pm.value ? 'PM' : 'AM';
})

watch(() => hourMinute.value, () => {
  debugLog('TimePicker.vue watch hourMinute.value: ', hourMinute.value);
  if (hourMinute.value) {
    pm.value = hourMinute.value.hour >= 12;
  }
})

watch(() => hourMinute.value?.hour, () => {
  if (hourMinute.value) {
    if (hourMinute.value.hour === 24) {
      hourMinute.value.hour = 0;
    }
    pm.value = hourMinute.value.hour >= 12;
  }
  debugLog('TimePicker.vue watch hourMinute.value.hour: ', hourMinute.value?.hour);
})

watch(() => hourMinute.value?.minute, () => {
  debugLog('TimePicker.vue watch hourMinute.value.minute: ', hourMinute.value?.minute);
})

watch(() => pm.value, () => {
  debugLog("TimePicker.vue watch pm.value: ", pm.value, hourMinute.value?.hour, ':',
      hourMinute.value?.minute);
  if (hourMinute.value) {
    const updated = {...hourMinute.value}
    debugLog(' if (hourMinute.value) ...');
    if (pm.value) {
      debugLog('if pm.value ...');
      if (hourMinute.value.hour < 12) {
        debugLog('if hourMinute.value.hour <= 12 ...');
        updated.hour += 12
      }
    } else {
      debugLog('else pm.value ...');
      if (hourMinute.value.hour >= 12) {
        debugLog('if hourMinute.value.hour >= 12 ...');
        updated.hour -= 12
      }
    }
    hourMinute.value = updated // 🛠️ Reassign to trigger reactivity
  }
})

watch(open, (val) => {
  selecting.value = 'hour'
})

function onClockSelect(value: number) {
  if (hourMinute.value) {
    const updated = {...hourMinute.value}
    debugLog('in if hourMinute.value: ', hourMinute.value);
    if (selecting.value === 'hour') {
      updated.hour = value
      if (pm.value && !is24h.value) {
        updated.hour += 12;
      }
      // selecting.value = 'minute'
    } else {
      updated.minute = value
      // selecting.value = 'hour'
    }
    hourMinute.value = updated // 🛠️ Reassign to trigger reactivity
    debugLog("TimePicker onClockSelect: ", hourMinute.value, 'selecting: ', selecting.value);
  }
}

function switchMode() {
  debugLog('TimePicker.vue switchMode: ', selecting.value);
  if (selecting.value === 'hour') {
    selecting.value = 'minute'
  } else {
    selecting.value = 'hour'
  }
}

function switchToMinutes() {
  debugLog('TimePicker.vue switchMode: ', selecting.value);
  if (selecting.value === 'hour') {
    selecting.value = 'minute'
  } else {
    open.value = false
  }
}

function onUpdateAmPm(value: boolean) {
  debugLog('TimePicker.vue onUpdateAmPm: ', value);
  pm.value = safeHour.value >= 12;
}

function confirm() {
  debugLog('TimePicker.vue emit update:hourMinute', {...localTime.value});
  emit('update:hourMinute', {...localTime.value})

}

function activeTabClass(tab: 'hour' | 'minute') {
  return tab === selecting.value
      ? 'text-neutral-100 text-5xl font-bold'
      : 'text-neutral-100 text-5xl'
}

</script>

<template>
  <div class="flex flex-col items-center justify-center mx-auto p-4">
    <!-- Input string + clock button + Popover -->
    <div class="flex items-center space-x-2">
      <!-- Input string -->
      <!--      <input type="text" placeholder="Enter time"-->
      <!--             class="border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"/>-->

      <div ref="wrapper" class="relative flex">
        <!-- Input string -->
        <!--        <input @click="togglePopover" type="text" placeholder="Click to open clock" v-bind="attrs" v-model="time"-->
        <!--               class="border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"/>-->
        <!-- Popover button with clock -->
        <button @click="togglePopover"
                class="w-10 h-10 flex items-center justify-center bg-blue-500 text-white rounded-md hover:bg-blue-600">
          <!-- lucide:clock-3 -->
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
            <g fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2">
              <circle cx="12" cy="12" r="10"/>
              <path d="M12 6v6h4.5"/>
            </g>
          </svg>
        </button>

        <!-- Popover content -->
        <div v-show="isOpen"
             class="absolute left-1/2 top-full mt-2 transform -translate-x-1/2 z-10  p-4  bg-gray-200 border shadow-lg">
          <div class="container mx-auto flex flex-col items-center justify-center">
            <!-- Time Display 24h -->
            <div v-if="is24h" :style="{ backgroundColor: primaryColor }"
                 class="flex flex-row items-center justify-center gap-4 text-neutral-100 p-2">
              <button @click="selecting = 'hour'" :class="activeTabClass('hour')">
                {{ localHour }}
              </button>
              <span class="text-5xl font-semibold text-neutral-600 ">:</span>
              <button @click="selecting = 'minute'" :class="activeTabClass('minute')">
                {{ paddedTime.minute }}
              </button>
            </div>
            <!-- Time Display AM/PM -->
            <div v-if="!is24h" :style="{ backgroundColor: primaryColor }"
                 class="flex flex-row items-center justify-center align-center gap-4 text-neutral-100 p-2">
              <button @click="selecting = 'hour'" :class="activeTabClass('hour')">
                {{ ampmHour }}
              </button>
              <span class="text-5xl font-semibold text-center align-middle text-neutral-600">:</span>
              <button @click="selecting = 'minute'" :class="activeTabClass('minute')">
                {{ paddedTime.minute }}
              </button>
              <button @click="selecting = 'minute'" class="text-2xl font-semibold">
                {{ pm ? 'PM' : 'AM' }}
              </button>
            </div>

            <!-- Format Switch -->
            <div class="flex flex-row gap-2 items-center justify-center p-2 m-2">
              <Switch class="w-30" v-model="is24h" label-enabled="24h" label-disabled="AM/PM"/>
              <!--            <div class="text-lg text-center w-15">{{ is24h ? '24h' : 'AM/PM' }}</div>-->
              <Checkbox v-if="!is24h" label-enabled="PM" label-disabled="AM" class="w-20" v-model="pm"/>
            </div>

            <div class="w256 h256">
              <!-- Clock Dial -->
              <ClockDial
                  :mode="selecting"
                  :hour="localHour"
                  :minute="localMinute"
                  :is24h="is24h"
                  :is-open="isOpen"
                  :pm="pm"
                  @update="onClockSelect"
                  @updatePm="onUpdateAmPm"
                  @switch="switchToMinutes"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

</template>

<style>
.text-5xl {
  font-size: 3rem; /* 48px */
  line-height: 1;
}

.font-semibold {
  font-weight: 600;
}

.text-2xl {
  font-size: 1.5rem; /* 24px */
  line-height: 2rem; /* 32px */
}

.font-bold {
  font-weight: 700;
}

.text-neutral-100 {
  color: #f5f5f5;
}

.text-neutral-100 {
  color: #f5f5f5;
}

.text-primary {
  color: oklch(69.6% 0.17 162.48);
}

.w256 {
  width: 256px;
}

.h256 {
  height: 256px;
}

</style>
