<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch } from 'vue';
import WorkReminder from './WorkReminder.vue';

const STORAGE_KEY = 'timeCalculator';
const currentTime = new Date();
const formattedCurrentTime = currentTime.toTimeString().substring(0, 5);

const startTime = ref(formattedCurrentTime);
const breakDuration = ref(30);
const workDuration = ref(8.5);
const remainingTime = ref('');
let timerInterval: number;

const loadSavedData = () => {
  const today = new Date().toDateString();
  const saved = localStorage.getItem(STORAGE_KEY);
  
  if (saved) {
    const data = JSON.parse(saved);
    if (data.date === today) {
      startTime.value = data.startTime;
      breakDuration.value = data.breakDuration;
      workDuration.value = data.workDuration;
    } else {
      // Clear old data if it's a new day
      localStorage.removeItem(STORAGE_KEY);
    }
  }
};

const saveData = () => {
  const data = {
    date: new Date().toDateString(),
    startTime: startTime.value,
    breakDuration: breakDuration.value,
    workDuration: workDuration.value
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
};

const endTime = computed(() => {
  const [hours, minutes] = startTime.value.split(':').map(Number);
  const date = new Date();
  date.setHours(hours, minutes);

  const workMinutes = Math.floor(workDuration.value * 60);
  const totalMinutes = workMinutes + breakDuration.value;
  date.setMinutes(date.getMinutes() + totalMinutes);

  return date.toTimeString().substring(0, 5);
});

const formattedWorkDuration = computed(() => {
  const hours = Math.floor(workDuration.value);
  const minutes = Math.round((workDuration.value - hours) * 60);
  return `${hours}h ${minutes}m`;
});

const updateRemainingTime = () => {
  const now = new Date();
  const [endHours, endMinutes] = endTime.value.split(':').map(Number);
  const end = new Date();
  end.setHours(endHours, endMinutes, 0);

  if (now > end) {
    remainingTime.value = 'Work day completed!';
    return;
  }

  const diff = end.getTime() - now.getTime();
  const hours = Math.floor(diff / (1000 * 60 * 60));
  const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
  const seconds = Math.floor((diff % (1000 * 60)) / 1000);

  remainingTime.value = `${hours}h ${minutes}m ${seconds}s`;
};

watch([startTime, breakDuration, workDuration], () => {
  saveData();
});

onMounted(() => {
  loadSavedData();
  timerInterval = setInterval(updateRemainingTime, 1000);
  updateRemainingTime();
});

onUnmounted(() => {
  clearInterval(timerInterval);
});
</script>

<template>
  <div>
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
          <div class="mt-4 p-4 bg-blue-50 rounded-md">
            <h3 class="text-xl font-medium text-blue-900">
              Time Remaining:
              <span class="text-blue-700">{{ remainingTime }}</span>
            </h3>
          </div>
        </div>
      </div>
    </div>
    
    <WorkReminder
      :start-time="startTime"
      :end-time="endTime"
    />
  </div>
</template>