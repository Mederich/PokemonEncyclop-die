<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'

// --- Interfaces (Mises à jour) ---

interface PokemonType {
  name: string;
  image: string;
}

interface PokemonSprites {
  regular: string;
  shiny?: string;
}

interface PokemonStats {
  hp: number;
  atk: number;
  def: number;
  spe_atk: number;
  spe_def: number;
  vit: number;
}

interface PokemonResistance {
  name: string;
  multiplier: number;
}

interface PokemonTalent {
  name: string;
  tc: boolean;
}

// NOUVEAU: Interfaces pour les évolutions
interface PokemonEvolutionInfo {
  pokedex_id: number;
  name: string;
  condition?: string; // Ex: "Niveau 16"
}

interface PokemonEvolution {
  pre: PokemonEvolutionInfo | null;
  next: PokemonEvolutionInfo[] | null;
  mega?: any; // On ignore les méga-évolutions pour l'instant
}

// Interface Pokemon principale (Mise à jour)
interface Pokemon {
  pokedex_id: number;
  generation: number;
  category: string;
  name: {
    fr: string;
    en: string;
    jp: string;
  };
  sprites: PokemonSprites;
  types: PokemonType[];
  talents: PokemonTalent[];
  stats: PokemonStats;
  resistances: PokemonResistance[];
  height: string;
  weight: string;
  evolution: PokemonEvolution; // <-- PROPRIÉTÉ AJOUTÉE
}

// --- Logique du Composant ---

const pokemons = ref<Pokemon[]>([])
const searchQuery = ref('')
const selectedPokemon = ref<Pokemon | null>(null)
const selectedTypes = ref<string[]>([])

onMounted(async () => {
  try {
    const response = await fetch('https://tyradex.vercel.app/api/v1/gen/1')
    const data: Pokemon[] = await response.json()
    pokemons.value = data
  } catch (error) {
    console.error("Erreur lors du chargement de la génération :", error)
  }
})

const availableTypes = computed(() => {
  const typesSet = new Set<string>()
  pokemons.value.forEach(pokemon => {
    pokemon.types.forEach(type => {
      typesSet.add(type.name)
    })
  })
  return Array.from(typesSet).sort()
})

const filteredPokemons = computed(() => {
  const query = searchQuery.value.toLowerCase().trim()
  const types = selectedTypes.value

  return pokemons.value.filter(pokemon => {
    const nameMatch = pokemon.name.fr.toLowerCase().includes(query)
    const idMatch = pokemon.pokedex_id.toString().includes(query)
    const textMatch = !query || (nameMatch || idMatch)

    const typeMatch = types.length === 0 ||
                      pokemon.types.some(type => types.includes(type.name))

    return textMatch && typeMatch
  })
})

function selectPokemon(pokemon: Pokemon) {
  selectedPokemon.value = pokemon
}

function clearSelection() {
  selectedPokemon.value = null
}

function clearTypeFilters() {
  selectedTypes.value = []
}

// NOUVEAU: Fonction pour obtenir le sprite d'une évolution par son ID
function getPokemonSprite(id: number): string {
  const evoPokemon = pokemons.value.find(p => p.pokedex_id === id);
  return evoPokemon ? evoPokemon.sprites.regular : ''; // Retourne le sprite ou une chaîne vide
}

// NOUVEAU: Fonction pour naviguer vers un autre Pokémon
function viewEvolution(id: number) {
  const targetPokemon = pokemons.value.find(p => p.pokedex_id === id);
  if (targetPokemon) {
    selectedPokemon.value = targetPokemon; // Change le Pokémon sélectionné
    window.scrollTo(0, 0); // Remonte en haut de la page
  }
}
</script>

<template>
  <div v-if="selectedPokemon" class="pokemon-detail-view">
    <button @click="clearSelection" class="back-button">Retour à la liste</button>

    <div class="detail-card">
      <img :src="selectedPokemon.sprites.regular" :alt="selectedPokemon.name.fr" class="detail-image" />
      <span class="detail-id">#{{ selectedPokemon.pokedex_id }}</span>
      <h1 class="detail-name">{{ selectedPokemon.name.fr }}</h1>
      <p class="detail-category">{{ selectedPokemon.category }}</p>

      <div class="detail-types">
        <span v-for="type in selectedPokemon.types" :key="type.name" class="type-badge">
          <img :src="type.image" :alt="type.name" class="type-image" />
          {{ type.name }}
        </span>
      </div>

      <div class="info-grid">
        <div><h3>Taille</h3><p>{{ selectedPokemon.height }}</p></div>
        <div><h3>Poids</h3><p>{{ selectedPokemon.weight }}</p></div>
      </div>

      <h3>Statistiques de base</h3>
      <div class="stats-grid">
        <span>HP: <strong>{{ selectedPokemon.stats.hp }}</strong></span>
        <span>Attaque: <strong>{{ selectedPokemon.stats.atk }}</strong></span>
        <span>Défense: <strong>{{ selectedPokemon.stats.def }}</strong></span>
        <span>Atk. Spé: <strong>{{ selectedPokemon.stats.spe_atk }}</strong></span>
        <span>Déf. Spé: <strong>{{ selectedPokemon.stats.spe_def }}</strong></span>
        <span>Vitesse: <strong>{{ selectedPokemon.stats.vit }}</strong></span>
      </div>

      <h3>Talents</h3>
      <div class="talents-grid">
        <span v-for="talent in selectedPokemon.talents" :key="talent.name">
          {{ talent.name }}
          <em v-if="talent.tc">(Talent Caché)</em>
        </span>
      </div>

      <h3>Résistances</h3>
      <div class="resistances-grid">
        <div v-for="res in selectedPokemon.resistances" :key="res.name" class="resistance-item">
          <span class="resistance-name">{{ res.name }}</span>
          <span :class="['resistance-value', res.multiplier >= 2 ? 'weak' : (res.multiplier < 1 ? 'resist' : '')]">
            x{{ res.multiplier }}
          </span>
        </div>
      </div>

      <div class="evolution-section">
        <h3>Évolutions</h3>

        <div v-if="!selectedPokemon.evolution.pre && !selectedPokemon.evolution.next">
          <p>Ce Pokémon n'a pas d'évolution dans cette génération.</p>
        </div>

        <div class="evolution-chain">
          <div v-if="selectedPokemon.evolution.pre"
               class="evolution-card"
               @click="viewEvolution(selectedPokemon.evolution.pre.pokedex_id)">
            <img :src="getPokemonSprite(selectedPokemon.evolution.pre.pokedex_id)" :alt="selectedPokemon.evolution.pre.name" />
            <span>#{{ selectedPokemon.evolution.pre.pokedex_id }} {{ selectedPokemon.evolution.pre.name }}</span>
            <small>(Évolue de)</small>
          </div>

          <div class="evolution-card current">
            <img :src="selectedPokemon.sprites.regular" :alt="selectedPokemon.name.fr" />
            <span>#{{ selectedPokemon.pokedex_id }} {{ selectedPokemon.name.fr }}</span>
            <small>(Actuel)</small>
          </div>

          <div v-if="selectedPokemon.evolution.next">
            <div v-for="evo in selectedPokemon.evolution.next"
                 :key="evo.pokedex_id"
                 class="evolution-card"
                 @click="viewEvolution(evo.pokedex_id)">
              <img :src="getPokemonSprite(evo.pokedex_id)" :alt="evo.name" />
              <span>#{{ evo.pokedex_id }} {{ evo.name }}</span>
              <small class="evolution-condition">{{ evo.condition }}</small>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>

  <div v-else>
    <h1>Pokédex - Génération 1</h1>

    <input
      v-model="searchQuery"
      type="text"
      placeholder="Rechercher par nom ou ID..."
      class="search-bar"
    />

    <div class="filter-container">
      <h3>Filtrer par type :</h3>
      <div class="type-filters">
        <label v-for="type in availableTypes" :key="type" class="type-filter-label">
          <input type="checkbox" :value="type" v-model="selectedTypes" />
          {{ type }}
        </label>
      </div>
      <button
        v-if="selectedTypes.length > 0"
        @click="clearTypeFilters"
        class="clear-filters-button"
      >
        Réinitialiser les types
      </button>
    </div>

    <div v-if="pokemons.length === 0">
      <p>Chargement des Pokémon...</p>
    </div>

    <div v-else class="">
      <div v-if="filteredPokemons.length === 0">
        <p>Aucun Pokémon ne correspond à vos filtres.</p>
      </div>

      <div v-else class="pokemon-list">
        <div
          v-for="pokemon in filteredPokemons"
          :key="pokemon.pokedex_id"
          class="pokemon-card"
          @click="selectPokemon(pokemon)"
        >
          <img :src="pokemon.sprites.regular" :alt="pokemon.name.fr" class="pokemon-image" />
          <span class="pokemon-id">#{{ pokemon.pokedex_id }}</span>
          <h2 class="pokemon-name">{{ pokemon.name.fr }}</h2>
          <div class="pokemon-types">
            <span v-for="type in pokemon.types" :key="type.name" class="type-badge">
              <img :src="type.image" :alt="type.name" class="type-image" />
              {{ type.name }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
/* ... (tous vos styles précédents) ... */

/* NOUVEAU: Styles pour la vue détaillée (Talents, Résistances, Évolutions) */
.talents-grid {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  background-color: #f9f9f9;
  padding: 15px;
  border-radius: 8px;
  margin: 10px 0 20px 0;
  text-align: center;
}
.talents-grid em {
  font-size: 0.8em;
  color: #666;
}

.resistances-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 10px;
  background-color: #f9f9f9;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  text-align: left;
}
.resistance-item {
  display: flex;
  justify-content: space-between;
  padding: 5px;
  border-bottom: 1px solid #eee;
}
.resistance-name {
  font-weight: 500;
}
.resistance-value {
  font-weight: bold;
}
.resistance-value.weak {
  color: #dc2626; /* Rouge (faiblesse) */
}
.resistance-value.resist {
  color: #16a34a; /* Vert (résistance) */
}

.evolution-section {
  background-color: #f9f9f9;
  padding: 15px;
  border-radius: 8px;
  margin-top: 20px;
}
.evolution-section h3 {
  text-align: center;
  margin-top: 0;
}
.evolution-section > p {
  text-align: center;
  color: #666;
  font-style: italic;
}

.evolution-chain {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-wrap: wrap;
  gap: 15px;
}

.evolution-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #ddd;
  background-color: #fff;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  min-width: 100px;
}
.evolution-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
.evolution-card.current {
  border-color: #0284c7; /* Bleu pour le Pokémon actuel */
  border-width: 2px;
  cursor: default;
}
.evolution-card.current:hover {
  transform: none;
  box-shadow: none;
}
.evolution-card img {
  width: 80px;
  height: 80px;
  object-fit: contain;
  background-color: #f1f1f1;
  border-radius: 50%;
}
.evolution-card span {
  font-size: 0.9em;
  font-weight: bold;
}
.evolution-card small {
  font-size: 0.8em;
  color: #555;
}
.evolution-condition {
  font-weight: normal !important;
  font-style: italic;
}

/* Styles généraux et de la liste */
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  background-color: #f4f5f7;
  color: #333;
  margin: 0;
  padding: 10px;
}
h1, h2, h3 {
  text-align: center;
}
.search-bar {
  display: block;
  width: 100%;
  max-width: 400px;
  margin: 20px auto;
  padding: 10px;
  font-size: 16px;
  border-radius: 8px;
  border: 1px solid #ccc;
}
.filter-container {
  max-width: 800px;
  margin: 20px auto;
  padding: 15px;
  background-color: #fff;
  border-radius: 8px;
  border: 1px solid #eee;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}
.filter-container h3 {
  margin-top: 0;
  margin-bottom: 10px;
  text-align: center;
}
.type-filters {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
}
.type-filter-label {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 5px 10px;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 20px;
  cursor: pointer;
  user-select: none;
  transition: background-color 0.2s;
}
.type-filter-label input[type="checkbox"] {
  margin-right: 5px;
}
.type-filter-label:has(input:checked) {
  background-color: #e0f2fe;
  border-color: #0284c7;
}
.clear-filters-button {
  display: block;
  margin: 15px auto 0 auto;
  background-color: #ef4444;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9em;
}
.clear-filters-button:hover {
  background-color: #dc2626;
}
.pokemon-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}
.pokemon-card {
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 16px;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #fff;
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.pokemon-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}
.pokemon-image {
  width: 120px;
  height: 120px;
  object-fit: contain;
  margin-bottom: 10px;
}
.pokemon-id {
  font-size: 0.9em;
  color: #888;
}
.pokemon-name {
  margin: 5px 0 10px 0;
  font-size: 1.4em;
  text-transform: capitalize;
}
.pokemon-types {
  display: flex;
  justify-content: center;
  gap: 5px;
  margin-top: auto;
}
.type-badge {
  display: flex;
  align-items: center;
  background-color: #f1f1f1;
  border-radius: 10px;
  padding: 3px 8px;
  font-size: 0.8em;
  border: 1px solid #ddd;
}
.type-image {
  width: 16px;
  height: 16px;
  margin-right: 4px;
}
.pokemon-detail-view {
  max-width: 700px;
  margin: 20px auto;
  padding: 20px;
}
.back-button {
  background-color: #f1f1f1;
  border: 1px solid #ddd;
  padding: 10px 15px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  margin-bottom: 20px;
}
.back-button:hover {
  background-color: #e9e9e9;
}
.detail-card {
  background-color: #fff;
  border: 1px solid #eee;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  padding: 24px;
}
.detail-image {
  width: 200px;
  height: 200px;
  object-fit: contain;
  margin: 0 auto 10px auto;
  display: block;
  background-color: #f8f8f8;
  border-radius: 50%;
}
.detail-id {
  font-size: 1.2em;
  color: #888;
  text-align: center;
  display: block;
}
.detail-name {
  font-size: 2.5em;
  text-transform: capitalize;
  margin: 5px 0;
  text-align: center;
}
.detail-category {
  font-size: 1.2em;
  color: #555;
  margin-bottom: 20px;
  text-align: center;
}
.detail-types {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}
.type-badge {
  display: flex;
  align-items: center;
  background-color: #f1f1f1;
  border-radius: 10px;
  padding: 5px 10px;
  font-size: 1em;
  border: 1px solid #ddd;
}
.type-image {
  width: 20px;
  height: 20px;
  margin-right: 5px;
}
.info-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  background-color: #f9f9f9;
  padding: 15px;
  border-radius: 8px;
  margin: 20px 0;
  text-align: center;
}
.info-grid h3 {
  margin: 0 0 5px 0;
  font-size: 1em;
  color: #777;
}
.info-grid p {
  margin: 0;
  font-size: 1.2em;
  font-weight: bold;
}
.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
  text-align: left;
  padding: 15px;
  background-color: #f9f9f9;
  border-radius: 8px;
  margin: 10px 0 20px 0;
}
.stats-grid span {
  font-size: 1.1em;
}
</style>