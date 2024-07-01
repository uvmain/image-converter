<script setup lang="ts">
import { ref, computed } from 'vue';

const imageUrl = ref<string | null>(null);
const imageSize = ref<number>(800);
const fileName = ref<string | undefined>('');
const imageUrlInput = ref<string>('');

function onFileChange(event: Event) {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  fileName.value = file?.name;
  if (file) {
    const reader = new FileReader();
    reader.onload = () => {
      imageUrl.value = reader.result as string;
      imageUrlInput.value = '';
    };
    reader.readAsDataURL(file);
  }
}

async function onUrlChange() {
  try {
    const response = await fetch(imageUrlInput.value);
    if (!response.ok) throw new Error('Network response was not ok');
    const blob = await response.blob();
    const reader = new FileReader();
    reader.onloadend = () => {
      imageUrl.value = reader.result as string;
      fileName.value = imageUrlInput.value.split('/').pop()?.split('?')[0];
    };
    reader.readAsDataURL(blob);
  } catch (error) {
    alert('Failed to load image. Please check the URL and try again.');
  }
}

function onPaste(event: ClipboardEvent) {
  const items = event.clipboardData?.items;
  if (items) {
    for (const item of items) {
      if (item.type.startsWith('image/')) {
        const file = item.getAsFile();
        if (file) {
          fileName.value = 'pasted_image';
          const reader = new FileReader();
          reader.onload = () => {
            imageUrl.value = reader.result as string;
            imageUrlInput.value = '';
          };
          reader.readAsDataURL(file);
        }
      }
    }
  }
}

function downloadImage(format: string) {
  if (!imageUrl.value) return;

  const img = new Image();
  img.src = imageUrl.value;
  img.onload = () => {
    const canvas = document.createElement('canvas');

    const maxDimension = Math.min(imageSize.value, Math.max(img.width, img.height));
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
  return encodeURI(fileName.value?.replaceAll(' ','_').toLocaleLowerCase() || '').replace(/\.[^/.]+$/, "");
});
</script>

<template>
  <div class="bg-light text-gray-700 min-h-screen min-w-screen" @paste="onPaste">
    <div class="flex flex-col items-center gap-10 p-6">
      <div class="flex flex-col gap-0">
        <input 
          type="file" 
          @change="onFileChange" 
          accept="image/*"
          id="fileInput"
          class="hidden"
        />
        <label 
          for="fileInput" 
          class="block text-sm text-gray-700 border-1 border-solid border-gray-300 rounded-lg cursor-pointer bg-gray-50 focus:outline-none p-2 text-center hover:bg-gray-200 w-28rem"
        >
          Browse
        </label>
        <span class="text-sm">
          or
        </span>
        <input 
          v-model="imageUrlInput"
          @change="onUrlChange"
          placeholder="Enter image URL"
          class="block w-28rem mt-2 p-2 border-1 border-solid border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-600"
        />
        <span class="text-sm">
          or
        </span>
        <textarea
          placeholder="Paste an image here"
          class="block w-28rem mt-2 p-2 border-1 border-solid border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-600"
        ></textarea>
        <span v-if="fileName">
          {{ fileName }}
        </span>
        <div v-if="imageUrl" class="mt-4">
          <img 
            :src="imageUrl" 
            alt="Uploaded Image" 
            class="rounded-lg shadow-md w-28rem object-contain"
          />
        </div>
      </div>
      <div class="w-28rem flex flex-col gap-4 w-md">
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
            <button @click="downloadImage('webp')" class="border-1 border-solid border-gray-300 bg-sky-300 text-gray-700 rounded-lg px-4 py-2 hover:bg-sky-400 outline-none focus:ring-2 focus:ring-sky-300">WebP</button>
            <button @click="downloadImage('jpeg')" class="border-1 border-solid border-gray-300 bg-lime-200 text-gray-700 rounded-lg px-4 py-2 hover:bg-lime-400 outline-none focus:ring-2 focus:ring-lime-300">JPEG</button>
            <button @click="downloadImage('png')" class="border-1 border-solid border-gray-300 bg-red-300 text-gray-700 rounded-lg px-4 py-2 hover:bg-red-400 outline-none focus:ring-2 focus:ring-red-300">PNG</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
