<template>
    <div class="container">
        <img
        alt="Pokemon Logo"
        src="../assets/pokeLogo.png"
        class="imgLogo"
        >
        <h2>A complete Dex with all generations</h2>

        <PokemonSearch 
            :api-url="apiUrl" 
            @setPokemonUrl="setPokemonUrl"
        />

        <PokemonList 
            :imageUrl="imageUrl" 
            :apiUrl="apiUrl"
            @setPokemonUrl="setPokemonUrl"
        />

        <PokemonDetail
            v-if="showDetail"
            :pokemonUrl="pokemonUrl"
            :imageUrl="imageUrl"
            @closeDetail="closeDetail"
        />
    </div>
</template>

<script>
import PokemonSearch from "./PokemonSearch.vue"
import PokemonList from "./PokemonList.vue"
import PokemonDetail from "./PokemonDetail.vue"

export default {
    data: () => {
        return {
            imageUrl: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/",
            pokemonUrl: "",
            apiUrl: "https://pokeapi.co/api/v2/pokemon/",
            showDetail: false
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
        }
    },
};
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  padding: 5px;
  width: calc(100% - 10px);
  min-height: calc(100vh - 10px);

  font-family: "Big Shoulders Display", cursive;
  font-size: 1rem;
}

h2 {
  color: #2944df;
}

@media(max-width: 768px) {
    h2 {
        text-align: center;
    }
}
@media(max-width: 524px) {
    .imgLogo {
        width: 300px;
    }
}
</style>