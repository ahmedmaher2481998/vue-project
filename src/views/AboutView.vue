<template>
  <div>
    <!-- The canvas is now in App.vue, so this view can be template-less -->
    <button @click="play" style="max-height: 100px; max-width: 100px;">Play</button>
    <!-- add clip  -->
    <button @click="addClip" style="max-height: 100px; max-width: 100px;">Add Clip</button>
    <button @click="exportTimeline" style="max-height: 100px; max-width: 100px;">Export Timeline</button>
    <div v-if="exportUrl" style="margin-top: 1em;">
      <a :href="exportUrl" download="timeline.mp4">Download Exported Timeline</a>
    </div>
  </div>
</template>

<script setup lang="ts">
import { Engine } from "@rendley/sdk";
import { ref } from "vue";
const randomAssests = ref(['https://videos.pexels.com/video-files/31688355/13500874_1080_1920_30fps.mp4',
  "https://images.pexels.com/photos/24253539/pexels-photo-24253539/free-photo-of-a-bridge-over-a-river-with-a-city-in-the-background.jpeg?auto=compress&cs=tinysrgb&w=500",
  "https://images.pexels.com/photos/31452622/pexels-photo-31452622.jpeg",
])
// Store the export URL for download link
const exportUrl = ref<string | null>(null);

const exportTimeline = async () => {
  const exportData = await Engine.getInstance().export();
  console.log(exportData);

  // Clean up previous URL if any
  if (exportUrl.value) {
    URL.revokeObjectURL(exportUrl.value);
    exportUrl.value = null;
  }

  // Create a Blob and object URL for download
  const blob = exportData?.blob;
  if (!blob) {
    console.error("No blob found in export data.");
    return;
  }
  exportUrl.value = URL.createObjectURL(blob);
}

// Track engine readiness
const isEngineReady = ref(false);

// Poll for engine readiness (since App.vue initializes it asynchronously)
function checkEngineReady() {
  try {
    // Check if Engine is initialized and timeline is available
    const engine = Engine.getInstance();
    if (
      engine &&
      engine.getTimeline &&
      typeof engine.getTimeline === "function" &&
      engine.getTimeline()
    ) {
      isEngineReady.value = true;
      return true;
    }
  } catch (e) {
    // Engine not ready yet
    console.error("Engine not ready:", e);
    return false;
  }
  return false;
}

// Try to check readiness every 200ms until ready
if (!checkEngineReady()) {
  const interval = setInterval(() => {
    if (checkEngineReady()) {
      clearInterval(interval);
    }
  }, 200);
}

const play = () => {
  if (!isEngineReady.value) {
    console.error("Engine is not ready. Cannot play timeline.");
    return;
  }
  try {
    const timeline = Engine.getInstance().getTimeline();
    if (!timeline) {
      console.error("Timeline is not available.");
      return;
    }
    if (timeline.getFitDuration() === 0) {
      console.warn("Timeline duration is zero. Nothing to play.");
      return;
    }
    timeline.play();
    console.log("Timeline duration:", timeline.getFitDuration());
    console.log("Playing");
  } catch (error) {
    console.error("Error while trying to play timeline:", error);
  }
};


const addClip = async () => {
  if (!isEngineReady.value) {
    console.error("Engine is not ready. Cannot add clip.");
    return;
  }
  try {
    const engine = Engine.getInstance();
    const timeline = engine.getTimeline();
    const library = engine.getLibrary();

    // Add media
    const mediaId = await library.addMedia(
      randomAssests.value.pop()!
    );
    if (!mediaId) {
      console.error("Failed to add media to library.");
      return;
    }
    console.log("Media added with ID:", mediaId);

    // Create layer and add clip
    const layer = Engine.getInstance().getTimeline().createLayer();
    const clipId = await layer.addClip({
      mediaDataId: mediaId,
      startTime: Engine.getInstance().getTimeline().getFitDuration(),
      duration: 5,
    });

    if (!clipId) {
      console.error("Failed to add clip to layer.");
      return;
    }

    // Confirm that the timeline has a non-zero duration
    const fitDuration = timeline.getFitDuration();
    if (fitDuration > 0) {
      console.log("Clip added. Timeline duration is now:", fitDuration);
    } else {
      console.warn("Clip added, but timeline duration is still zero.");
    }

    console.log("Clip ID:", clipId);

  } catch (error) {
    console.error("Error adding clip to timeline:", error);
  }
};
</script>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
