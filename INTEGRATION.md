# Integrating Rendley SDK with Vue.js

This guide explains how to integrate the Rendley SDK (built with Stencil.js) into a Vue.js application, based on our implementation experience.

## Key Implementation Considerations

### FFmpeg Loading Challenge

One critical issue we encountered during implementation was related to FFmpeg initialization. The Rendley SDK heavily relies on FFmpeg for video rendering and manipulation, and proper initialization is crucial.

**Important Note**: The FFmpeg WASM file is substantial in size and requires:

- Download time: 3-10 seconds (varies based on internet speed)
- Proper initialization handling
- Loading state management

### Core Integration Steps

1. **Root Component Setup (Main App)**

```typescript
// App.vue
import { Engine, StorageIndexedDB, AutoLayoutMode } from '@rendley/sdk'

async function initializeRendley() {
  // Wait for FFmpeg to load - This is crucial!
  // add error ignore for tsc because it's expecting values for the ffmpeg file location but we need to apply the default values
  await Engine.getInstance().getFFmpeg().load()

  // Initialize engine only after FFmpeg is ready
  Engine.getInstance().init({
    display: {
      width: 500,
      height: 500,
      backgroundColor: '#0000',
      view: canvas,
    },
    storages: [new StorageIndexedDB()],
  })
}
```

2. **Loading State Management**

```typescript
// Track loading state
const isLoaded = ref(false);

// Update state after FFmpeg loads
await Engine.getInstance().getFFmpeg().load();
isLoaded.value = true;

// Only render content when ready
<template>
  <div>
    <LoadingIndicator v-if="!isLoaded" />
    <RouterView v-else />
  </div>
</template>
```

3. **Canvas/Editor Page Implementation**

```typescript
// Editor component
import { Engine } from '@rendley/sdk'

// Ensure engine is ready before operations
const isEngineReady = ref(false)

function checkEngineReady() {
  try {
    const engine = Engine.getInstance()
    if (engine && engine.getTimeline()) {
      isEngineReady.value = true
      return true
    }
  } catch (e) {
    return false
  }
  return false
}

// Example timeline operation
const addClip = async () => {
  if (!isEngineReady.value) {
    console.error('Engine not ready')
    return
  }

  const engine = Engine.getInstance()
  const timeline = engine.getTimeline()
  // Proceed with clip addition
}
```

## Best Practices

### 1. Initialization

- Always wait for FFmpeg to load completely before initializing the engine
- Implement proper loading states
- Handle initialization errors gracefully

### 2. Engine Instance Management

- Use Engine.getInstance() consistently
- Check engine readiness before operations
- Maintain proper initialization order

### 3. Resource Management

- Handle media assets properly
- Manage timeline and clip operations carefully
- Implement proper cleanup

## Common Pitfalls

1. **FFmpeg Loading**

   - Not waiting for FFmpeg to load completely
   - Missing loading state implementation
   - Attempting operations before initialization

2. **Engine Operations**
   - Accessing engine methods before ready
   - Missing error handling
   - Improper timeline management

## Implementation Example

Our implementation follows this structure:

```
src/
├── App.vue                 # Core initialization
└── views/
    └── Editor.vue         # Canvas/editing implementation
```

### Key Features Implemented

1. **Video Manipulation**

   - Timeline management
   - Clip addition/removal
   - Preview functionality
   - Export capabilities

2. **User Interface**
   - Loading indicators
   - Control buttons
   - Preview canvas
   - Export options

## Troubleshooting

### FFmpeg Loading Issues

If FFmpeg fails to load:

1. Check network connectivity
2. Verify WASM support in the browser
3. Ensure proper initialization sequence
4. Check for console errors

### Engine Initialization

If engine fails to initialize:

1. Verify FFmpeg loaded successfully
2. Check canvas element availability
3. Verify license configuration
4. Review initialization parameters

## Conclusion

Successful integration of Rendley SDK with Vue.js requires careful attention to:

- FFmpeg loading sequence
- Engine initialization
- State management
- Error handling

Following these guidelines will help avoid common pitfalls and ensure a smooth integration process.
