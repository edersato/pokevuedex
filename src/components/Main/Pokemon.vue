<template>
    <div class="container">
        <img
        alt="Pokemon Logo"
        src="../assets/pokeLogo.png"
        class="imgLogo"
        >
        <h2>A complete Dex with all generations</h2>

        <div class="region-selector">
            <label for="region">Região / Region:</label>
            <select id="region" v-model="selectedRegion" @change="onRegionChange">
                <option v-for="region in regions" :key="region.id" :value="region.id">
                    {{ region.name }} ({{ region.start }} - {{ region.end }})
                </option>
            </select>
        </div>

        <PokemonSearch 
            :api-url="apiUrl" 
            @setPokemonUrl="setPokemonUrl"
        />

        <PokemonList 
            :imageUrl="imageUrl" 
            :apiUrl="apiUrl"
            :region="selectedRegion"
            @setPokemonUrl="setPokemonUrl"
            @changeRegion="changeRegion"
        />

        <PokemonDetail
            v-if="showDetail"
            :pokemonUrl="pokemonUrl"
            :imageUrl="imageUrl"
            @closeDetail="closeDetail"
            @selectPokemon="setPokemonUrl"
        />
    </div>
</template>

<script>
import PokemonSearch from "../Search/PokemonSearch.vue"
import PokemonList from "../List/PokemonList.vue"
import PokemonDetail from "../Details/PokemonDetail.vue"
import { regions } from "../../data/region.js"

export default {
    data: () => {
        return {
            imageUrl: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/",
            pokemonUrl: "",
            apiUrl: "https://pokeapi.co/api/v2/pokemon/",
            showDetail: false,
            regions: regions,
            selectedRegion: 10 // Começa com "Todas"
        };
    },
    components: {
        PokemonSearch,
        PokemonList,
        PokemonDetail
    },
    methods: {
        setPokemonUrl(url) {
            this.pokemonUrl = url;
            this.showDetail = true;
        },

        closeDetail() {
            this.pokemonUrl = "";
            this.showDetail = false;
        },

        onRegionChange() {
            // Quando muda a região, não precisamos resetar nada
            // O PokemonList já vai lidar com a mudança
            console.log('Região alterada para:', this.selectedRegion);
        },
        
        changeRegion(regionId) {
            this.selectedRegion = regionId;
        }
    },
};
</script>

<style lang="scss" scoped src="./index.scss" />