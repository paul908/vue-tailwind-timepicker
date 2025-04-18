<script setup lang="ts">
import {ref, computed, watch, onMounted, nextTick, useAttrs, onBeforeUnmount} from 'vue'
import Checkbox from "./Checkbox.vue";
import ClockDial from "./ClockDial.vue";
import Switch from "./Switch.vue";

/**
 * InputTimePicker component
 * @description A time picker component that allows users to select a time using a clock dial.
 * @author Paul Becue BQ Systems http://bqsystems.be
 * @version 1.0.0
 */

/**
 * @typedef {Object} TimePickerProps
 * @property {string} time - The selected time in HH:mm format.
 * @property {boolean} is24h - Flag to indicate if the time format is 24-hour or 12-hour (AM/PM).
 */
const time = defineModel<string>('time')
const is24h = defineModel<boolean>('is24h')


// capture "extra" attributes like inputClass, variant, etc.
const attrs = useAttrs()

/**
 * @description Debugging flag to enable or disable debug logs.
 * @type {boolean}
 */
const DEBUG = false;
function debugLog(...args: any) {
  if (DEBUG) console.log(...args);
}

/**
 * @description Formats the given hour and minute into a string in HH:mm format.
 * @param {number} hour - The hour to format.
 * @param {number} minute - The minute to format.
 * @returns {string} The formatted time string.
 */
function formatTime(hour: number, minute: number): string {
  const hh = hour.toString().padStart(2, '0');
  const mm = minute.toString().padStart(2, '0');
  // debugLog('InputTimePicker formatTime: ', hh, ':', mm, `${hh}:${mm}`);
  return `${hh}:${mm}`;
}

/**
 * @description Sets the local time based on the provided time string.
 * @param {string | undefined} time - The time string in HH:mm format.
 */
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

/**
 * @description Reactive references and DOM elements used in the InputTimePicker component.
 * These variables manage the state and behavior of the time picker.
 */

const selecting = ref<'hour' | 'minute'>('hour')
// Tracks the currently selected mode in the time picker (either 'hour' or 'minute').

const localHour = ref(0)
// Stores the local hour value selected by the user.

const localMinute = ref(0)
// Stores the local minute value selected by the user.

const pm = ref(false)
// Indicates whether the time is in PM (true) or AM (false) when using 12-hour format.

const isOpen = ref<boolean>(false)
// Tracks whether the time picker popover is open (true) or closed (false).

const wrapper = ref<any>(null)
// A reference to the wrapper DOM element for the time picker, used for click detection.

const canvasRef = ref<HTMLCanvasElement | null>(null);
// A reference to the ClockDial component, used for rendering and interaction.

setLocalTime(time.value);

/**
 * The primaryColor, neutralColor, and textColor variables are used to store the computed CSS colors
 * for the component's styling.
 */
// let primaryColor = getComputedStyle(document.documentElement).getPropertyValue('--color-blue-500').trim();
// let neutralColor = getComputedStyle(document.documentElement).getPropertyValue('--color-gray-400').trim();
// let textColor = getComputedStyle(document.documentElement).getPropertyValue('--color-neutral-50').trim();

/**
 * togglePopover is a function that toggles the visibility of the popover.
 * It sets the isOpen variable to true if it is currently false, and vice versa.
 */
function togglePopover() {
  isOpen.value = !isOpen.value
}

/**
 * handleClickOutside is an event handler that closes the popover if the user clicks outside of it.
 * It checks if the click target is not contained within the wrapper element.
 * @param event - The click event object.
 */
function handleClickOutside(event: any) {
  if (wrapper.value && !wrapper.value.contains(event.target)) {
    isOpen.value = false
  }
}

/**
 * onMounted lifecycle hook that adds a click event listener to the document when the component is mounted.
 * It also sets up a watcher to redraw the clock dial when the popover is opened.
 */
onMounted(async () => {
  document.addEventListener('click', handleClickOutside)
  debugLog("InputTimePicker onMounted: ", wrapper.value);

  // Watcher to redraw the clock dial when the popover is opened. Why the nextTick?
  // It was because otherwise you see no dial when you open the popover.
  watch(isOpen, async (visible) => {
    if (visible) {
      await nextTick();
      debugLog("after next tick: ");
      (canvasRef.value as any)?.draw?.();
      debugLog("after canvasRef.value?.draw?.();");
    }
  })
})

/**
 * onBeforeUnmount lifecycle hook that removes the click event listener from the document when the component is unmounted.
 */
onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside)
})

/**
 * safeIs24h is a computed property that returns the value of the is24h prop or defaults to true if not provided.
 * It is used to determine whether to display the time in 24-hour format or 12-hour format.
 */
const safeIs24h = computed(() => {
  // debugLog('TimePicker.vue safeIs24h: ', is24h.value ?? true);
  return is24h.value ?? true
})

/**
 * paddedTime is a computed property that returns an object containing the padded hour and minute values.
 * It ensures that the hour and minute are always displayed with two digits (e.g., "09" instead of "9").
 */
const paddedTime = computed(() => ({
  hour: localHour.value.toString().padStart(2, '0'),
  minute: localMinute.value.toString().padStart(2, '0')
}))

/**
 * ampmHour is a computed property that converts the local hour to 12-hour format if the is24h prop is false.
 * It ensures that the hour is displayed correctly in AM/PM format.
 */
const ampmHour = computed(() => {
  let x = localHour.value;
  if (x > 12) {
    x -= 12;
  } else if (x === 0 || x === 24) {
    x = 12;
  }
  return x
})

/**
 * watch function that monitors changes to the time prop and updates the local hour and minute values accordingly.
 * It also sets the pm variable based on the hour value.
 */
watch(() => time.value, () => {
  // debugLog('InputTimePicker watch(() => time.value: ', time.value);
  setLocalTime(time.value);
})

/**
 * watch function that monitors changes to the pm variable and updates the local hour value accordingly.
 * It adjusts the hour value based on whether the time is in AM or PM.
 */
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

/**
 * watch function that monitors changes to the isOpen variable and sets the selecting variable to 'hour' when the popover is opened.
 */
watch(isOpen, (val) => {
  debugLog('val: ', val);
  selecting.value = 'hour'
})

/**
 * onClockSelect is an event handler that is called when the user selects a value from the clock dial.
 * It updates the local hour or minute based on the selected value and formats the time accordingly.
 * @param value - The selected value from the clock dial (hour or minute).
 */
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

/**
 * switchMode is a function that toggles the selecting variable between 'hour' and 'minute'.
 * It is used to switch between hour and minute selection modes in the popover.
 */
// function switchMode() {
//   if (selecting.value === 'hour') {
//     selecting.value = 'minute'
//   } else {
//     selecting.value = 'hour'
//   }
//   debugLog('TimePicker.vue switchMode: ', selecting.value);
// }

/**
 * switchToMinutes is a function that switches the selecting variable to 'minute' if it is currently 'hour'.
 * If it is already 'minute', it closes the popover.
 */
function switchToMinutes() {
  if (selecting.value === 'hour') {
    selecting.value = 'minute'
  } else {
    isOpen.value = false;
  }
  debugLog('TimePicker.vue switchToMinutes: ', selecting.value);
}

/**
 * onUpdateAmPm is an event handler that is called when the user updates the AM/PM selection.
 * It updates the pm variable based on the selected value and adjusts the local hour accordingly.
 * @param value - The selected AM/PM value (true for PM, false for AM).
 */
function onUpdateAmPm(value: boolean) {
  debugLog('InputTimePicker.vue onUpdateAmPm: ', value);
  pm.value = localHour.value >= 12;
}

/**
 * activeTabClass is a utility function that returns the appropriate CSS class for the active tab (hour or minute).
 * It is used to style the hour and minute buttons in the popover.
 * @param tab - The tab to check ('hour' or 'minute').
 * @returns The CSS class for the active tab.
 */
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
             class="absolute left-1/2 top-full mt-2 transform -translate-x-1/2 z-10  bg-gray-200 border rounded-xl shadow-lg">
          <div class="container mx-auto flex flex-col items-center justify-center">
            <div class="popover-header rounded-xl">
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
                         :is24h="safeIs24h"
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
