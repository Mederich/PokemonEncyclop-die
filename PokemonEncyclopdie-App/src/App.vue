<script setup lang="ts"></script>

<template>
  <div class="min-h-screen flex flex-col items-center justify-center bg-gray-100">
    <h1 class="text-4xl font-bold text-blue-600 mb-4">🎉 Tailwind est bien là !</h1>
    <p class="text-gray-700 text-lg">
      Si ce texte est bien <span class="text-green-600 font-semibold">vert</span> et qu’il y a un fond gris clair,<br />
      alors Tailwind CSS fonctionne correctement ✅
    </p>

    <button
      class="mt-6 px-6 py-2 rounded-lg bg-gradient-to-r from-indigo-500 to-purple-600 text-white hover:scale-105 transition-transform"
    >
      Bouton test Tailwind
    </button>

    <p class="mt-10 text-sm text-gray-500">
      Consulte <a href="https://tailwindcss.com/docs" class="text-blue-500 underline" target="_blank">la doc Tailwind</a>
    </p>
  </div>
</template>

<style scoped></style>
