<script setup lang="ts">
import { ref, computed } from 'vue';

const currentTime = new Date();
const formattedCurrentTime = currentTime.toTimeString().substring(0, 5);

const startTime = ref(formattedCurrentTime);
const breakDuration = ref(30);
const workDuration = ref(8.5); // 8.5 hours = 8 hours and 30 minutes

const endTime = computed(() => {
  const [hours, minutes] = startTime.value.split(':').map(Number);
  const date = new Date();
  date.setHours(hours, minutes);

  // Convert decimal hours to total minutes and add break duration
  const workMinutes = Math.floor(workDuration.value * 60);
  const totalMinutes = workMinutes + breakDuration.value;
  date.setMinutes(date.getMinutes() + totalMinutes);

  return date.toTimeString().substring(0, 5);
});

// Format work duration for display
const formattedWorkDuration = computed(() => {
  const hours = Math.floor(workDuration.value);
  const minutes = Math.round((workDuration.value - hours) * 60);
  return `${hours}h ${minutes}m`;
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
        <label for="workDuration" class="block text-sm font-medium text-gray-700">
          Work Duration (hours)
        </label>
        <input
          type="number"
          id="workDuration"
          v-model="workDuration"
          min="0"
          max="24"
          step="0.25"
          class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        />
        <p class="text-sm text-gray-500 mt-1">
          Use decimals for partial hours (e.g., 8.5 for 8h 30m)
        </p>
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
          {{ formattedWorkDuration }} of work
        </p>
      </div>
    </div>
  </div>
</template>