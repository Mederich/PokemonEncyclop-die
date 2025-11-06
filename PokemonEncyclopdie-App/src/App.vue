<script setup lang="ts">
import { ref, onMounted } from 'vue'
 
interface PokemonType {
  name: string;
  image: string;
}
 
interface Pokemon {
  pokedex_id: number;
  name: {
    fr: string;
  };
  types: PokemonType[];
}
 
const pokemons = ref<Pokemon[]>([])
 
onMounted(async () => {
  try {
    const response = await fetch('https://tyradex.vercel.app/api/v1/gen/1')
    const data: Pokemon[] = await response.json()
    pokemons.value = data
  } catch (error) {
    console.error("Erreur lors du chargement de la génération :", error)
  }
})
</script>
 
<template>
  <h1>Pokédex - Génération 1</h1>
 
  <div v-if="pokemons.length === 0">
    <p>Chargement des Pokémon...</p>
  </div>
 
  <div v-else class="">
    <div v-for="pokemon in pokemons" :key="pokemon.pokedex_id" class="border flex flex-row">
      
      <span class="pl-4 px-2">#{{ pokemon.pokedex_id }}</span>
 
      <h2 class="px-3">{{ pokemon.name.fr }}</h2>
      <div class="">
        <span v-for="type in pokemon.types" :key="type.name" class="px-1">
          {{ type.name }}
        </span>
      </div>
    </div>
  </div>
</template>
 
 