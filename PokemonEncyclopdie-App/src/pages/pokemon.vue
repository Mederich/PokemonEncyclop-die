<script setup lang="ts">
import { ref } from 'vue';

const pokemonId = ref<number | string>(1);
const pokemonData = ref<any>(null);
const loading = ref(false);
const error = ref<string | null>(null);

async function fetchPokemon(id: number | string) {
  loading.value = true;
  error.value = null;
  pokemonData.value = null;
  try {
    const res = await fetch(`https://tyradex.vercel.app/api/v1/pokemon/${id}`);
    if (!res.ok) throw new Error(`Erreur réseau : ${res.status}`);
    const data = await res.json();
    pokemonData.value = data;
  } catch (err: any) {
    error.value = "Impossible de récupérer les données du Pokémon.";
    console.error(err);
  } finally {
    loading.value = false;
  }
}

fetchPokemon(pokemonId.value);
</script>

<template>
  <div class="min-h-screen bg-zinc-100 text-zinc-900 font-mono p-6">
    <div class="max-w-4xl mx-auto bg-zinc-50 border border-zinc-300 rounded-lg shadow-lg overflow-hidden">
      <!-- En-tête -->
      <div class="bg-zinc-200 border-b border-zinc-400 flex justify-between items-center px-6 py-3">
        <h1 class="text-2xl font-bold uppercase tracking-widest text-zinc-800">
          Pokédex Encyclopédique
        </h1>
        <div class="flex gap-2">
          <div class="w-3 h-3 rounded-full bg-green-400"></div>
          <div class="w-3 h-3 rounded-full bg-yellow-400"></div>
          <div class="w-3 h-3 rounded-full bg-red-500"></div>
        </div>
      </div>

      <div class="p-6">
        <!-- Barre de recherche -->
        <div class="flex gap-2 mb-6">
          <input
            v-model="pokemonId"
            type="text"
            placeholder="Nom ou ID du Pokémon"
            class="flex-1 px-3 py-2 rounded border border-zinc-400 bg-zinc-100 focus:ring-1 focus:ring-zinc-500"
          />
          <button
            @click="fetchPokemon(pokemonId)"
            class="bg-zinc-800 hover:bg-zinc-700 text-zinc-100 px-4 py-2 rounded font-semibold uppercase"
          >
            Chercher
          </button>
        </div>

        <!-- États -->
        <div v-if="loading" class="text-center text-zinc-600 italic">Chargement...</div>
        <div v-if="error" class="text-center text-red-500">{{ error }}</div>

        <!-- Données du Pokémon -->
        <div
          v-if="pokemonData"
          class="grid md:grid-cols-2 gap-6 bg-zinc-100 border border-zinc-300 rounded-lg p-6"
        >
          <!-- Image -->
          <div class="flex flex-col items-center justify-center">
            <div class="relative bg-zinc-200 border border-zinc-400 rounded-md p-4 shadow-inner">
              <img
                v-if="pokemonData.sprites?.regular"
                :src="pokemonData.sprites.regular"
                alt="Sprite Pokémon"
                class="w-48 h-48 object-contain"
              />
              <div class="absolute inset-0 border border-zinc-400 rounded-md opacity-40"></div>
            </div>

            <!-- Types -->
            <div class="mt-4 flex flex-wrap justify-center gap-3">
              <div
                v-for="type in pokemonData.types"
                :key="type.name"
                class="flex items-center gap-1 px-3 py-1 rounded-full border border-zinc-400 bg-white shadow-sm"
              >
                <img :src="type.image" alt="" class="w-5 h-5" />
                <span class="font-semibold text-zinc-800">{{ type.name }}</span>
              </div>
            </div>
          </div>

          <!-- Infos -->
          <div>
            <h2 class="text-3xl font-bold mb-1 tracking-wide text-zinc-800">
              {{ pokemonData.name?.fr || pokemonData.name?.en }}
            </h2>
            <p class="text-zinc-500 mb-3 italic">
              Pokédex #{{ pokemonData.pokedex_id }}
            </p>

            <div class="space-y-1 text-zinc-700 mb-4">
              <p><span class="font-semibold">Taille :</span> {{ pokemonData.height }} m</p>
              <p><span class="font-semibold">Poids :</span> {{ pokemonData.weight }} kg</p>
              <p><span class="font-semibold">Génération :</span> {{ pokemonData.generation }}</p>
            </div>

            <!-- Statistiques -->
            <div class="mt-4">
              <h3 class="font-semibold text-lg mb-2 text-zinc-800 border-b border-zinc-400">
                Statistiques
              </h3>
              <div
                v-for="(value, key) in pokemonData.stats"
                :key="key"
                class="mb-2"
              >
                <div class="flex justify-between text-sm">
                  <span class="capitalize">{{ key }}</span>
                  <span>{{ value }}</span>
                </div>
                <div class="h-2 bg-zinc-300 rounded overflow-hidden">
                  <div
                    class="h-2 bg-gradient-to-r from-yellow-400 to-green-500"
                    :style="{ width: (value / 255 * 100) + '%' }"
                  ></div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <!-- /PokemonData -->
      </div>
    </div>
  </div>
</template>
