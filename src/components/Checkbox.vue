<script setup lang="ts">
import { computed } from 'vue'

const checked = defineModel()

const props = defineProps<{
  labelEnabled: string
  labelDisabled: string
}>()

const enabled = computed(() => {
  return checked.value
})

</script>

<template>
  <label class="inline-flex items-center space-x-2 cursor-pointer">
    <input
        type="checkbox"
        v-model="checked"
        class="w-5 h-5 text-blue-600 bg-white border-gray-300 rounded focus:ring-blue-500 focus:ring-2"
    />
    <span v-if="enabled" class="text-gray-900">
    {{ props.labelEnabled }}
</span>
    <span v-else class="text-gray-900">
    {{ props.labelDisabled }}
</span>
  </label>
</template>

