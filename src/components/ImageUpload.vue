<script setup lang="ts">

const imageUrl = ref<string | null>(null);
const imageSize = ref<number>(800);

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
  <div class="flex flex-col gap-10 bg-zinc-900 text-light p-4">
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
          class="rounded-lg shadow-md max-w-800px"
        />
      </div>
    </div>
    <div class="flex flex-col gap-4">
      <input
        v-model="imageSize"
        type="range"
        min="100"
        max="2000"
        step="100"
        class="w-full"
      >
      <div>
        Maximum width/height: {{ imageSize }}px
      </div>
      <div class="flex justify-center">
        <div class="grid grid-cols-2 grid-rows-2 gap-4 w-48">
          <button class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300 text-dark">WebP</button>
          <button class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300 text-dark">JPEG</button>
          <button class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300 text-dark">PNG</button>
          <button class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300 text-dark">AVIF</button>
        </div>
      </div>
    </div>
  </div>
</template>
