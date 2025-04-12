<script setup lang="ts">
/**
 * Switch component
 * @description A simple switch component that toggles between two states.
 * @example <Switch labelEnabled="On" labelDisabled="Off" />
 * @param {string} labelEnabled - The label for the enabled state.
 * @param {string} labelDisabled - The label for the disabled state.
 */

const enabled = defineModel()

const props = defineProps<{
  labelEnabled: string
  labelDisabled: string
}>()

</script>

<template>
  <label class="relative inline-flex items-center cursor-pointer">
    <input
        type="checkbox"
        class="sr-only peer"
        v-model="enabled"
    />
    <div
        class="w-11 h-6 bg-gray-300 peer-focus:outline-none peer-focus:ring-2 peer-focus:ring-blue-500 rounded-full peer peer-checked:bg-blue-600 transition-colors"
    ></div>
    <div
        class="absolute left-1 top-1 bg-white w-4 h-4 rounded-full transition-transform peer-checked:translate-x-5"
    ></div>
    <span v-if="enabled" class="ml-3 text-sm font-medium text-gray-900">
      {{ props.labelEnabled }}
    </span>
    <span v-else class="ml-3 text-sm font-medium text-gray-900">
      {{ props.labelDisabled }}
    </span>
  </label>
</template>

