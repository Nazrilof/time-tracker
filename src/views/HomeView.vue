<script setup lang="ts">
import { onBeforeUnmount, onMounted, reactive, ref } from 'vue';
import type { Ref } from 'vue';

interface Timer {
  id: number;
  project: string;
  time: number;
}

let currentTimer: Ref<number|null> = ref(null);

let timers: Timer[] = reactive([]);

let interval: number;

let showCreateModal = ref(false);

let createModalProject = ref('');

onMounted(() => {
  console.log('HomeView mounted');

  if (localStorage.getItem('timers')) {
    const storedTimers = JSON.parse(localStorage.getItem('timers')!);

    storedTimers.forEach((timer: Timer) => {
      timers.push(timer);
    });
  }

  interval = setInterval(() => {
    if (currentTimer.value !== null)
      timers[currentTimer.value].time++;

    localStorage.setItem('timers', JSON.stringify(timers));
  }, 1000);
});

onBeforeUnmount(() => {
  console.log('HomeView beforeUnmount');

  clearInterval(interval);
});

const formatTime = (time: number) => {
  const hours = String(Math.floor(time / 3600)).padStart(2, '0');
  const minutes = String(Math.floor((time % 3600) / 60)).padStart(2, '0');
  const seconds = String(time % 60).padStart(2, '0');

  return `${hours}:${minutes}:${seconds}`;
};

const handleCreateProject = () => {
  if (!createModalProject.value) return;

  timers.push({
    id: timers.length + 1,
    project: createModalProject.value,
    time: 0,
  });

  showCreateModal.value = false;
  createModalProject.value = '';
};

const handleDeleteProject = (i: number) => {
  console.log('delete', i);

  if (currentTimer.value === i) 
    currentTimer.value = null;

  timers.splice(i, 1);
};

</script>

<template>

  <main>
    <div class="current-timer">
      <template v-if="currentTimer !== null">
        <p>{{ timers[currentTimer].project }}</p>
        <p class="timer">{{ formatTime(timers[currentTimer].time) }}</p>
      </template>
      <p v-else>No timer running</p>
      <button @click="currentTimer = null" :disabled="currentTimer === null">stop</button>
    </div>

    <h2 class="saved-header">
      Saved Timers
      <button class="green" @click="showCreateModal = true">add</button>
    </h2>
    <ul class="saved-timers">
      <li v-for="(timer, i) in timers" :key="i">
        <span class="project">{{ timer.project }}</span>
        <span class="timer">{{ formatTime(timer.time) }} </span>
        <button @click="currentTimer = i" :disabled="currentTimer === i">start</button>
        <button @click="() => handleDeleteProject(i)">delete</button>
      </li>
      <p class="no-timers" v-if="timers.length === 0">no timers</p>
    </ul>
  </main>

  <Teleport to="body">
    <div class="modal-filter" v-if="showCreateModal" @click.self="showCreateModal = false">
      <div class="modal">
        <h2>Create Project</h2>
        <input type="text" placeholder="Project Name" v-model="createModalProject" />
        <button class="btn-create green" @click="handleCreateProject">Create</button>
        <button class="btn-cancel" @click="showCreateModal = false">Cancel</button>
      </div>
    </div>
  </Teleport>

</template>

<style lang="scss">
.modal-filter {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1;
  overflow: auto;
  display: grid;
  justify-content: center;
  align-items: center;

  .modal {
    background-color: var(--color-background);
    padding: 2rem;
    border-radius: 5px;
    z-index: 2;

    display: grid;

    grid-template-columns: 1fr auto auto;
    grid-template-rows: auto auto 1fr auto;
    grid-gap: 1rem;

    input { grid-row: 2; grid-column: 1 / -1; }

    button { grid-row: 4; }
    .btn-cancel { grid-column: 2; }
    .btn-create { grid-column: 3; }
  }
}

.current-timer {
  border: 1px solid var(--color-border);
  border-radius: 5px;
  padding: 1rem;

  display: grid;
  grid-template-columns: 1fr auto auto;
  grid-gap: 1rem;

  align-items: center;

  .timer {
    color: darken(lightgray, 50%);
  }

  button {
    grid-column: 3;
  }
}

.saved-header {
  display: flex;
  justify-content: space-between;
  align-items: center;

  margin-top: 2rem;
  margin-bottom: 1rem;
}

.saved-timers {
  list-style: none;
  padding: 0;

  li {
    display: grid;
    align-items: center;

    grid-template-columns: 1fr auto auto auto;
    grid-gap: 1rem;

    margin-bottom: 0.5rem;

    .timer {
      color: darken(lightgray, 50%);
    }
  }

  .no-timers {
    text-align: center;
    color: darken(lightgray, 50%);
  }
}

@media (max-width: 1024px) {
  .modal-filter {
    align-items: end;

    .modal {
      width: 100vw;
      height: 90vh;
      padding: 1.5rem;
      padding-bottom: 3rem;

      border-radius: 12px 12px 0 0;
    }
  }
}
</style>
