<script setup lang="ts">
import {ref, computed, watch, onMounted, nextTick, useAttrs, onBeforeUnmount} from 'vue'
import Checkbox from "./Checkbox.vue";
import ClockDial from "./ClockDial.vue";
import Switch from "./Switch.vue";

const time = defineModel<string>('time')
const is24h = defineModel<boolean>('is24h')

// capture "extra" attributes like inputClass, variant, etc.
const attrs = useAttrs()

const DEBUG = true;

function debugLog(...args: any) {
  if (DEBUG) console.log(...args);
}

function formatTime(hour: number, minute: number): string {
  const hh = hour.toString().padStart(2, '0');
  const mm = minute.toString().padStart(2, '0');
  // debugLog('InputTimePicker formatTime: ', hh, ':', mm, `${hh}:${mm}`);
  return `${hh}:${mm}`;
}

const setLocalTime = (time: string | undefined) => {
  debugLog('InputTimePicker setLocalTime => input time: ', time);
  const strValue: string = time ?? '00:00';
  debugLog('InputTimePicker setLocalTime => time.value: ', strValue);
  const [hhStr, mmStr] = strValue.split(":");
  const hh = parseInt(hhStr, 10);
  const mm = parseInt(mmStr, 10);
  pm.value = hh >= 12;
  if (hh === 24) {
    localHour.value = 0;
  } else {
    localHour.value = hh;
  }
  localMinute.value = mm;
}

const selecting = ref<'hour' | 'minute'>('hour')
const localHour = ref(0)
const localMinute = ref(0)
const pm = ref(false)

const isOpen = ref<boolean>(false)
const wrapper = ref<any>(null)
const canvasRef = ref(null)

setLocalTime(time.value);

let primaryColor = getComputedStyle(document.documentElement).getPropertyValue('--color-blue-500').trim();
let neutralColor = getComputedStyle(document.documentElement).getPropertyValue('--color-gray-400').trim();
let textColor = getComputedStyle(document.documentElement).getPropertyValue('--color-neutral-50').trim();


function togglePopover() {
  isOpen.value = !isOpen.value
}

function handleClickOutside(event: any) {
  if (wrapper.value && !wrapper.value.contains(event.target)) {
    isOpen.value = false
  }
}

onMounted(async () => {
  document.addEventListener('click', handleClickOutside)
  debugLog("InputTimePicker onMounted: ", wrapper.value);

  watch(isOpen, async (visible) => {
    if (visible) {
      await nextTick();
      debugLog("after next tick: ");
      canvasRef.value?.draw?.();
      debugLog("after canvasRef.value?.draw?.();");
    }
  })
})

onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside)
})

const safeIs24h = computed(() => {
  // debugLog('TimePicker.vue safeIs24h: ', is24h.value ?? true);
  return is24h.value ?? true
})

const paddedTime = computed(() => ({
  hour: localHour.value.toString().padStart(2, '0'),
  minute: localMinute.value.toString().padStart(2, '0')
}))

const ampmHour = computed(() => {
  let x = localHour.value;
  if (x > 12) {
    x -= 12;
  } else if (x === 0 || x === 24) {
    x = 12;
  }
  return x
})

watch(() => time.value, () => {
  // debugLog('InputTimePicker watch(() => time.value: ', time.value);
  setLocalTime(time.value);
})

watch(() => pm.value, () => {
  if (pm.value) {
    if (localHour.value < 12) {
      localHour.value += 12
    }
  } else {
    if (localHour.value >= 12) {
      localHour.value -= 12
    }
  }
  if (localHour.value === 24) {
    localHour.value = 0;
  }
  time.value = formatTime(localHour.value, localMinute.value);
  // debugLog("InputTimePicker.vue watch pm.value: ", pm.value, localHour.value, ':', localMinute.value,
  //     ' - time.value: ', time.value);
})

watch(isOpen, (val) => {
  debugLog('val: ', val);
  selecting.value = 'hour'
})

function onClockSelect(value: number) {
  debugLog("InputTimePicker onClockSelect: ", value);
  if (selecting.value === 'hour') {
    localHour.value = value
    if (pm.value && !is24h.value) {
      localHour.value += 12;
    }
    // selecting.value = 'minute'
  } else {
    localMinute.value = value
    // selecting.value = 'hour'
  }
  time.value = formatTime(localHour.value, localMinute.value);
  debugLog("InputTimePicker onClockSelect: ", localHour.value, ':', localMinute.value, 'time: ', time.value);
}

function switchMode() {
  if (selecting.value === 'hour') {
    selecting.value = 'minute'
  } else {
    selecting.value = 'hour'
  }
  debugLog('TimePicker.vue switchMode: ', selecting.value);
}

function switchToMinutes() {
  if (selecting.value === 'hour') {
    selecting.value = 'minute'
  } else {
    isOpen.value = false;
  }
  debugLog('TimePicker.vue switchToMinutes: ', selecting.value);
}

function onUpdateAmPm(value: boolean) {
  debugLog('InputTimePicker.vue onUpdateAmPm: ', value);
  pm.value = localHour.value >= 12;
}

function activeTabClass(tab: 'hour' | 'minute') {
  return tab === selecting.value
      ? 'text-neutral-100 font-bold text-5xl'
      : 'text-neutral-300 text-5xl'
}


</script>

<template>
  <div class="flex flex-col items-center justify-center mx-auto">
    <!-- Input string + clock button + Popover -->
    <div class="flex items-center space-x-2">
      <!-- Input string -->
      <!--      <input type="text" placeholder="Enter time"-->
      <!--             class="border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"/>-->

      <div ref="wrapper" class="relative flex">
        <!-- Input string -->
        <input @click="togglePopover" type="text" placeholder="Click to open clock" v-bind="attrs" v-model="time"
               class=" w-20 border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500 text-center"/>
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
             class="absolute left-1/2 top-full mt-2 transform -translate-x-1/2 z-10  bg-gray-200 border shadow-lg">
          <div class="container mx-auto flex flex-col items-center justify-center">
            <div class="popover-header">
              <!-- Time Display 24h -->
              <div v-if="is24h"
                   class="flex flex-row items-center justify-center gap-4 text-neutral-200 p-2">
                <button @click="selecting = 'hour'" :class="activeTabClass('hour')">
                  {{ localHour }}
                </button>
                <span class="text-5xl font-semibold text-neutral-200 ">:</span>
                <button @click="selecting = 'minute'" :class="activeTabClass('minute')">
                  {{ paddedTime.minute }}
                </button>
              </div>
              <!-- Time Display AM/PM -->
              <div v-if="!is24h"
                   class="flex flex-row items-center justify-center align-center gap-4 text-neutral-200 p-2">
                <button @click="selecting = 'hour'" :class="activeTabClass('hour')">
                  {{ ampmHour }}
                </button>
                <span class="text-5xl font-semibold text-center align-middle text-neutral-200">:</span>
                <button @click="selecting = 'minute'" :class="activeTabClass('minute')">
                  {{ paddedTime.minute }}
                </button>
                <button @click="selecting = 'minute'" class="text-2xl font-semibold">
                  {{ pm ? 'PM' : 'AM' }}
                </button>
              </div>
            </div>

            <!-- Format Switch -->
            <div class="flex flex-row gap-2 items-center justify-center p-2 m-2">
              <Switch class="w-30" v-model="is24h" label-enabled="24h" label-disabled="AM/PM"/>
              <!--            <div class="text-lg text-center w-15">{{ is24h ? '24h' : 'AM/PM' }}</div>-->
              <Checkbox v-if="!is24h" label-enabled="PM" label-disabled="AM" class="w-20" v-model="pm"/>
            </div>

            <!--            <div class="w256 h256">-->
            <div class="inline-block">
              <!-- Clock Dial -->
              <ClockDial ref="canvasRef"
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

<style scoped>
.popover-header {
  background-color: var(--color-blue-500); /* Replace with your desired color */
  color: var(--color-white);
  width: 100%; /* Ensures it spans the full width of the popover */
  padding: 8px; /* Optional: Add padding for spacing */
  box-sizing: border-box; /* Ensures padding is included in the width */
}
</style>
