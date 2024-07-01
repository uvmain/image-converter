<script setup lang="ts">
import { ref } from 'vue';

const imageUrl = ref<string | null>(null);
const imageSize = ref<number>(800);
const fileName = ref<string | undefined>('');

function onFileChange(event: Event) {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  fileName.value = file?.name
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
})
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
      {{ outputFileName }}
      <div class="flex justify-center">
        <div class="flex flex-row gap-4">
          <button @click="downloadImage('webp')" class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300 text-dark">WebP</button>
          <button @click="downloadImage('jpeg')" class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300 text-dark">JPEG</button>
          <button @click="downloadImage('png')" class="bg-gray-200 rounded-lg p-2 hover:bg-gray-300 text-dark">PNG</button>
        </div>
      </div>
    </div>
  </div>
</template>
