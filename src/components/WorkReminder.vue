<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue';

const STORAGE_KEY = 'workReminder';

const props = defineProps<{
  startTime: string;
  endTime: string;
}>();

const workInterval = ref(25);
const breakInterval = ref(5);
const isEnabled = ref(false);
const isBreak = ref(false);
const countdown = ref('');
let timer: number;
let countdownTimer: number;
const audio = new Audio('data:audio/wav;base64,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qJrx4KOQjahZxWbcZgztj2c49nKmkId44S71j0c8eV9yDK6uPRzx5X18eDvjvQ6yKo9ZSS6l//8elePK/Lf//IInrOF/FvDoADYAGBMGb7FtErm5MXMlmPAJQVgWta7Zx2go+8xJ0UiCb8LHHdftWyLJE0QIAIsI+UbXu67dZMjmgDGCGl1H+vpF4NSDckSIkk7Vd+sxEhBQMRU8j/12UIRhzSaUdQ+rQU5kGeFxm+hb1oh6pWWmv3uvmReDl0UnvtapVaIzo1jZbf/pD6ElLqSX+rUmOQNpJFa/r+sa4e/pBlAABoAAAAA3CUgShLdGIxsY7AUABPRrgCABdDuQ5GC7DqPQCgbbJUAoRSUj+NIEig0YfyWUho1VBBBA//uQZB4ABZx5zfMakeAAAAmwAAAAF5F3P0w9GtAAACfAAAAAwLhMDmAYWMgVEG1U0FIGCBgXBXAtfMH10000EEEEEECUBYln03TTTdNBDZopopYvrTTdNa325mImNg3TTPV9q3pmY0xoO6bv3r00y+IDGid/9aaaZTGMuj9mpu9Mpio1dXrr5HERTZSmqU36A3CumzN/9Robv/Xx4v9ijkSRSNLQhAWumap82WRSBUqXStV/YcS+XVLnSS+WLDroqArFkMEsAS+eWmrUzrO0oEmE40RlMZ5+ODIkAyKAGUwZ3mVKmcamcJnMW26MRPgUw6j+LkhyHGVGYjSUUKNpuJUQoOIAyDvEyG8S5yfK6dhZc0Tx1KI/gviKL6qvvFs1+bWtaz58uUNnryq6kt5RzOCkPWlVqVX2a/EEBUdU1KrXLf40GoiiFXK///qpoiDXrOgqDR38JB0bw7SoL+ZB9o1RCkQjQ2CBYZKd/+VJxZRRZlqSkKiws0WFxUyCwsKiMy7hUVFhIaCrNQsKkTIsLivwKKigsj8XYlwt/WKi2N4d//uQRCSAAjURNIHpMZBGYiaQPSYyAAABLAAAAAAAACWAAAAApUF/Mg+0aohSIRobBAsMlO//Kk4soosy1JSFRYWaLC4qZBYWFRGZdwqKiwkNBVmoWFSJkWFxX4FFRQWR+LsS4W/rFRb/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////VEFHAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAU291bmRib3kuZGUAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAwNGh0dHA6Ly93d3cuc291bmRib3kuZGUAAAAAAAAAACU=');

const loadSavedData = () => {
  const today = new Date().toDateString();
  const saved = localStorage.getItem(STORAGE_KEY);
  
  if (saved) {
    const data = JSON.parse(saved);
    if (data.date === today) {
      workInterval.value = data.workInterval;
      breakInterval.value = data.breakInterval;
      isEnabled.value = data.isEnabled;
    } else {
      // Clear old data if it's a new day
      localStorage.removeItem(STORAGE_KEY);
    }
  }
};

const saveData = () => {
  const data = {
    date: new Date().toDateString(),
    workInterval: workInterval.value,
    breakInterval: breakInterval.value,
    isEnabled: isEnabled.value
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
};

const requestNotificationPermission = async () => {
  if (!("Notification" in window)) {
    alert("This browser does not support notifications");
    return;
  }
  
  const permission = await Notification.requestPermission();
  if (permission !== "granted") {
    alert("Please allow notifications to use the reminder feature");
    isEnabled.value = false;
  }
};

const formatTime = (seconds: number) => {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${mins}:${secs.toString().padStart(2, '0')}`;
};

const updateCountdown = (endTime: number) => {
  const now = Date.now();
  const remaining = Math.max(0, Math.ceil((endTime - now) / 1000));
  countdown.value = formatTime(remaining);

  if (remaining === 0) {
    clearInterval(countdownTimer);
    audio.play();
    isBreak.value = !isBreak.value;
    if (isEnabled.value) {
      startNextInterval();
    }
  }
};

const startNextInterval = () => {
  const duration = isBreak.value ? breakInterval.value : workInterval.value;
  const endTime = Date.now() + duration * 60 * 1000;
  
  new Notification(isBreak.value ? "Break Time!" : "Back to Work!", {
    body: isBreak.value 
      ? `Take a ${breakInterval.value}-minute break to stand up and stretch`
      : `Time to focus for ${workInterval.value} minutes`,
    icon: "/vite.svg"
  });

  countdownTimer = window.setInterval(() => updateCountdown(endTime), 1000);
  updateCountdown(endTime);
};

const startTimer = () => {
  if (!isEnabled.value) return;
  
  const now = new Date();
  const [endHours, endMinutes] = props.endTime.split(':').map(Number);
  const endDate = new Date();
  endDate.setHours(endHours, endMinutes, 0);

  if (now >= endDate) {
    isEnabled.value = false;
    return;
  }

  isBreak.value = false;
  startNextInterval();
};

const stopTimer = () => {
  clearInterval(countdownTimer);
  countdown.value = '';
  isBreak.value = false;
};

watch([isEnabled, workInterval, breakInterval], () => {
  saveData();
});

watch(isEnabled, (newValue) => {
  if (newValue) {
    requestNotificationPermission().then(() => {
      if (Notification.permission === "granted") {
        startTimer();
      }
    });
  } else {
    stopTimer();
  }
});

watch([() => props.startTime, () => props.endTime], () => {
  if (isEnabled.value) {
    stopTimer();
    startTimer();
  }
});

onMounted(() => {
  loadSavedData();
});

onUnmounted(() => {
  stopTimer();
});
</script>

<template>
  <div class="bg-white rounded-lg shadow-lg p-6 mt-6">
    <h2 class="text-xl font-semibold text-gray-900 mb-4">Work Break Reminders</h2>
    
    <div class="space-y-4">
      <div class="flex items-center justify-between">
        <label class="text-sm font-medium text-gray-700">Enable Reminders</label>
        <button
          @click="isEnabled = !isEnabled"
          :class="[
            'px-4 py-2 rounded-md font-medium',
            isEnabled
              ? 'bg-green-500 text-white hover:bg-green-600'
              : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
          ]"
        >
          {{ isEnabled ? 'Enabled' : 'Disabled' }}
        </button>
      </div>

      <div v-if="isEnabled" class="text-center py-4">
        <div class="text-3xl font-bold" :class="{
          'text-blue-600': !isBreak,
          'text-green-600': isBreak
        }">
          {{ countdown }}
        </div>
        <div class="text-sm text-gray-600 mt-1">
          {{ isBreak ? 'Break Time' : 'Work Time' }}
        </div>
      </div>

      <div class="space-y-2">
        <label for="workInterval" class="block text-sm font-medium text-gray-700">
          Work Interval (minutes)
        </label>
        <input
          type="number"
          id="workInterval"
          v-model="workInterval"
          min="1"
          max="120"
          class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        />
      </div>

      <div class="space-y-2">
        <label for="breakInterval" class="block text-sm font-medium text-gray-700">
          Break Duration (minutes)
        </label>
        <input
          type="number"
          id="breakInterval"
          v-model="breakInterval"
          min="1"
          max="30"
          class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        />
      </div>
    </div>

    <p class="mt-4 text-sm text-gray-600">
      You'll receive notifications every {{ workInterval }} minutes to take a
      {{ breakInterval }}-minute break until your workday ends at {{ endTime }}.
    </p>
  </div>
</template>