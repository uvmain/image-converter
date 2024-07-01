<script setup lang="ts">
import { ref, computed } from 'vue';

const imageUrl = ref<string | null>(null);
const imageSize = ref<number>(800);
const fileName = ref<string | undefined>('');

function onFileChange(event: Event) {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  fileName.value = file?.name;
  if (file) {
    const reader = new FileReader();
    reader.onload = () => {
      imageUrl.value = reader.result as string;
    };
    reader.readAsDataURL(file);
  }
}

function downloadImage(format: string) {
  if (!imageUrl.value) return;

  const img = new Image();
  img.src = imageUrl.value;
  img.onload = () => {
    const canvas = document.createElement('canvas');
    const maxDimension = Math.min(imageSize.value, img.width, img.height);
    const aspectRatio = img.width / img.height;
    if (aspectRatio > 1) {
      canvas.width = maxDimension;
      canvas.height = maxDimension / aspectRatio;
    } else {
      canvas.height = maxDimension;
      canvas.width = maxDimension * aspectRatio;
    }

    const ctx = canvas.getContext('2d');
    if (ctx) {
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      canvas.toBlob((blob) => {
        if (blob) {
          const link = document.createElement('a');
          link.href = URL.createObjectURL(blob);
          link.download = `${outputFileName.value}.${format}`;
          link.click();
        }
      }, `image/${format}`);
    }
  };
}

const outputFileName = computed(() => {
  return encodeURI(fileName.value?.replaceAll(' ','_').toLocaleLowerCase() || '').replace(/\.[^/.]+$/, "")
});
</script>

<template>
  <div class="bg-light text-gray-700 min-h-screen min-w-screen">
    <div class="flex flex-col items-center gap-10 p-6">
      <div class="w-md flex flex-col gap-2">
        <input 
          type="file" 
          @change="onFileChange" 
          accept="image/*"
          id="fileInput"
          class="hidden"
        />
        <label 
          for="fileInput" 
          class="block text-sm text-gray-700 border-1 border-solid border-gray-300 rounded-lg cursor-pointer bg-gray-50 focus:outline-none p-2 text-center hover:bg-gray-200"
        >
          Browse
        </label>
        <div v-if="fileName">
          {{ fileName }}
        </div>
        <div v-if="imageUrl" class="mt-4">
          <img 
            :src="imageUrl" 
            alt="Uploaded Image" 
            class="rounded-lg shadow-md w-full object-contain"
          />
        </div>
      </div>
      <div class="w-full flex flex-col gap-4 w-md">
        <input
          v-model="imageSize"
          type="range"
          min="100"
          max="2000"
          step="100"
          class="w-full appearance-none bg-gray-300 rounded-lg h-2 focus:outline-none focus:ring-2 focus:ring-blue-600"
        >
        <div class="text-center">
          <span class="text-gray-400">
            Maximum width/height:
          </span>
          <span class="text-gray-700">
            {{ imageSize }}px
          </span>
        </div>
        <div class="flex justify-center mt-4">
          <div class="flex flex-row gap-4">
            <button @click="downloadImage('webp')" class="bg-blue-600 text-white rounded-lg px-4 py-2 hover:bg-blue-700 outline-none focus:ring-2 focus:ring-blue-500">WebP</button>
            <button @click="downloadImage('jpeg')" class="bg-green-600 text-white rounded-lg px-4 py-2 hover:bg-green-700 outline-none focus:ring-2 focus:ring-green-500">JPEG</button>
            <button @click="downloadImage('png')" class="bg-red-600 text-white rounded-lg px-4 py-2 hover:bg-red-700 outline-none focus:ring-2 focus:ring-red-500">PNG</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
