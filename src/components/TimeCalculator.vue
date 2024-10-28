<script setup lang="ts">
import { ref, computed } from 'vue';

const currentTime = new Date();
const formattedCurrentTime = currentTime.toTimeString().substring(0, 5);

const startTime = ref(formattedCurrentTime);
const breakDuration = ref(30);
const workDuration = ref(8);

const endTime = computed(() => {
  const [hours, minutes] = startTime.value.split(':').map(Number);
  const date = new Date();
  date.setHours(hours, minutes);

  // Add work hours plus break duration
  date.setMinutes(
    date.getMinutes() + workDuration.value * 60 + breakDuration.value
  );

  return date.toTimeString().substring(0, 5);
});
</script>

<template>
  <div class="bg-white rounded-lg shadow-lg p-6">
    <div class="space-y-6">
      <!-- Start Time Input -->
      <div class="space-y-2">
        <label for="startTime" class="block text-sm font-medium text-gray-700">
          Start Time
        </label>
        <input
          type="time"
          id="startTime"
          v-model="startTime"
          class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        />
      </div>

      <!-- Work Duration Input -->
      <div class="space-y-2">
        <label
          for="workDuration"
          class="block text-sm font-medium text-gray-700"
        >
          Work Duration (hours)
        </label>
        <input
          type="number"
          id="workDuration"
          v-model="workDuration"
          min="1"
          max="24"
          class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        />
      </div>

      <!-- Break Duration Input -->
      <div class="space-y-2">
        <label
          for="breakDuration"
          class="block text-sm font-medium text-gray-700"
        >
          Break Duration (minutes)
        </label>
        <input
          type="number"
          id="breakDuration"
          v-model="breakDuration"
          min="0"
          max="120"
          class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        />
      </div>

      <!-- Result -->
      <div class="mt-8 pt-6 border-t border-gray-200">
        <h2 class="text-2xl font-semibold text-gray-900">
          End Time: <span class="text-green-500">{{ endTime }}</span>
        </h2>
        <p class="mt-2 text-sm text-gray-600">
          Including {{ breakDuration }} minutes break after
          {{ workDuration }} hours of work
        </p>
      </div>
    </div>
  </div>
</template>
