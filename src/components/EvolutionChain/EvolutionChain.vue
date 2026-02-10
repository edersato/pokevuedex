<template>
  <div class="evolution-chain">
    <h3>Linha Evolutiva / Evolution Chain</h3>
    <div v-if="loading" class="loading">
      <i class="fas fa-spinner fa-spin"></i>
    </div>
    <div v-else-if="evolutionChain.length > 0" class="chain">
      <div 
        v-for="(evolution, index) in evolutionChain" 
        :key="evolution.id" 
        class="evolution-stage"
      >
        <div class="evolution-pokemon" @click="$emit('selectPokemon', evolution.id)">
          <img :src="imageUrl + evolution.id + '.png'" :alt="evolution.name">
          <span class="pokemon-name">{{ evolution.name }}</span>
          <span class="pokemon-id">#{{ evolution.id }}</span>
        </div>
        <div v-if="index < evolutionChain.length - 1" class="evolution-arrow">
          <i class="fas fa-arrow-right"></i>
          <span v-if="evolution.trigger" class="trigger">{{ evolution.trigger }}</span>
        </div>
      </div>
    </div>
    <div v-else class="no-evolution">
      Este Pokémon não evolui / This Pokémon does not evolve
    </div>
  </div>
</template>

<script>
export default {
  props: {
    speciesUrl: String,
    imageUrl: String
  },
  data() {
    return {
      evolutionChain: [],
      loading: true
    }
  },
  methods: {
    async fetchEvolutionChain() {
      this.loading = true;
      
      try {
        // Primeiro pega os dados da espécie
        const speciesResponse = await fetch(this.speciesUrl);
        const speciesData = await speciesResponse.json();
        
        // Pega a cadeia de evolução
        const chainResponse = await fetch(speciesData.evolution_chain.url);
        const chainData = await chainResponse.json();
        
        // Processa a cadeia de evolução
        this.evolutionChain = this.processChain(chainData.chain);
      } catch (error) {
        console.error('Erro ao buscar cadeia evolutiva:', error);
        this.evolutionChain = [];
      } finally {
        this.loading = false;
      }
    },
    
    processChain(chain, result = []) {
      // Pega o ID do Pokémon do URL
      const id = chain.species.url.split('/').filter(Boolean).pop();
      
      result.push({
        id: parseInt(id),
        name: chain.species.name,
        trigger: chain.evolution_details[0]?.trigger?.name || null
      });
      
      // Processa as evoluções recursivamente
      if (chain.evolves_to.length > 0) {
        chain.evolves_to.forEach(evolution => {
          this.processChain(evolution, result);
        });
      }
      
      return result;
    }
  },
  watch: {
    speciesUrl: {
      immediate: true,
      handler() {
        if (this.speciesUrl) {
          this.fetchEvolutionChain();
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped src="./index.scss" />