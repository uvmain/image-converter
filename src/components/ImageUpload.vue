<script setup lang="ts">
import { ref} from 'vue'
const imageUrl = ref<string | null>(null);
const inputSize = ref<number>(800);

function onFileChange(event: Event) {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = () => {
      imageUrl.value = reader.result as string;
    };
    reader.readAsDataURL(file);
  }
}
</script>

<template>
  <div class="flex flex-row gap-10">
    <div>
      <input 
        type="file" 
        @change="onFileChange" 
        accept="image/*"
        class="block w-full text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50 focus:outline-none"
      />
      <div v-if="imageUrl" class="mt-4">
        <img 
          :src="imageUrl" 
          alt="Uploaded Image" 
          class="rounded-lg shadow-md"
          :style="{ maxWidth: `${inputSize}px`, maxHeight: `${inputSize}px` }"
        />
      </div>
    </div>
    <div class="flex flex-col gap-4">
      <input
          v-model="inputSize"
          type="range"
          min="100"
          max="2000"
          step="100"
          class="w-100%"
        >
      <div>
      </div>
      <div class="grid grid-cols-2 grid-rows-2 h-20 gap-4">
        <button class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300">WebP</button>
        <button class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300">JPEG</button>
        <button class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300">PNG</button>
        <button class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300">AVIF</button>
      </div>
    </div>
  </div>
</template>