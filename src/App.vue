<script setup lang="ts">
import { Engine, StorageIndexedDB, AutoLayoutMode } from '@rendley/sdk';
import { nextTick, onBeforeMount, ref } from 'vue';
const isLoaded = ref(false);
async function initializeRendley() {
  try {
    const canvas = document.getElementById("myCanvas") as HTMLCanvasElement;
    if (!canvas) {
      console.error("Canvas element not found!");
      return;
    }

    // FFmpeg should be accessed via the Engine instance
    // @ts-expect-error - TODO: fix this
    await Engine.getInstance().getFFmpeg().load({
      // coreURL: "https://cdn.rendley.com/sdk/ffmpeg/1.0.0/dynamic/ffmpeg-core.js",
      // wasmURL: "https://cdn.rendley.com/sdk/ffmpeg/1.0.0/dynamic/ffmpeg-core.wasm",
      // workerURL: "https://unpkg.com/@ffmpeg/core@0.12.4/dist/ffmpeg-core.worker.js",
    });

    console.log("FFmpeg loaded");
    isLoaded.value = true;
    Engine.getInstance().init({
      license: {
        licenseName: "asd",
        licenseKey: "481E09D0F0EA4B19507B0001",
      },
      display: {
        width: 500,
        height: 500,
        backgroundColor: "#0000",
        view: canvas,
      },
      storages: [new StorageIndexedDB()],
    });

    // Apply additional settings seen in the StencilJS component
    Engine.getInstance().getSettings().setViewAutoLayoutOnResize(AutoLayoutMode.PREFER_HEIGHT);
    Engine.getInstance().getUndoManager().setEnabled(true);

    console.log("Rendley Engine Initialized Successfully");

    // The StencilJS component uses a `RendleyBridge` to load assets (filters, effects, etc.)
    // from CDN paths. This is custom logic. To replicate it, you would fetch the
    // asset JSON files and register them with the engine's library.

  } catch (error) {
    console.error("Error initializing Rendley SDK:", error);
  }
}

onBeforeMount(() => {
  // Use nextTick to ensure the DOM is fully settled before initialization
  nextTick(async () => {
    await initializeRendley();
  });
});
</script>

<template>
  <canvas id="myCanvas" style="width: 500px; height: 500px; border: 1px solid #444;"></canvas>
  <h1 v-if="!isLoaded">
    loading: {{ isLoaded }}
  </h1>
  <RouterView v-if="isLoaded" />
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
