<template>
  <div class="detail" @click.self="closeDetail">
    <div class="detail-view card" v-if="show">
      <!-- Botão de fechar no topo -->
      <button class="close-btn" @click="closeDetail">
        <i class="fas fa-times"></i>
      </button>
      
      <div v-if="pokemon" class="image">
        <img :src="getCurrentImage()" alt="">
      </div>

      <div v-if="pokemon" class="data card-body">
        <h2 class="card-title">{{ formatName(pokemon.name) }}</h2>
        
        <!-- Navegação entre variantes -->
        <div class="variants-nav" v-if="hasVariants">
          <button 
            v-for="variant in variants" 
            :key="variant.name"
            @click="selectVariant(variant)"
            :class="{ 'active': currentVariant === variant.name }"
            class="variant-btn"
          >
            {{ variant.displayName }}
          </button>
        </div>

        <!-- Informações básicas -->
        <div class="basic-info">
          <div class="property">
            <div class="left destaq">Base Exp/ Experiência Base</div>
            <div class="right">{{ pokemon.base_experience || 'N/A' }} XP</div>
          </div>
        
          <div class="property">
            <div class="left destaq">Altura/Height</div>
            <div class="right">{{ (pokemon.height / 10).toFixed(1) }} m</div>
          </div>

          <div class="property">
            <div class="left destaq">Peso/Weight</div>
            <div class="right">{{ (pokemon.weight / 10).toFixed(1) }} kg</div>
          </div>
        </div>

        <!-- Tipos -->
        <h3>Tipo/Type</h3>
        <div class="types">
          <div class="type" 
            v-for="(value, index) in pokemon.types"
            :key="'type-'+index">
            <span :class="value.type.name">
              {{ formatName(value.type.name) }}
            </span>
          </div>
        </div>

        <!-- Habilidades -->
        <h3>Habilidades/Abilities</h3>
        <div class="abilities">
          <div class="ability" 
            v-for="(value, index) in pokemon.abilities"
            :key="'ability-'+index">
            {{ formatName(value.ability.name) }}
            <span v-if="value.is_hidden" class="hidden-ability">(Hidden)</span>
          </div>
        </div>

        <!-- Linha Evolutiva -->
        <div v-if="evolutionChain.length > 0" class="evolution-section">
          <h3>Linha Evolutiva / Evolution Chain</h3>
          <div class="evolution-chain">
            <div 
              v-for="(stage, index) in evolutionChain" 
              :key="stage.id" 
              class="evolution-stage"
            >
              <div class="evolution-pokemon" @click="loadPokemonById(stage.id)">
                <img :src="imageUrl + stage.id + '.png'" :alt="stage.name">
                <span class="pokemon-name">{{ formatName(stage.name) }}</span>
                <span class="pokemon-id">#{{ stage.id }}</span>
              </div>
              <div v-if="index < evolutionChain.length - 1" class="evolution-arrow">
                <i class="fas fa-arrow-right"></i>
                <span v-if="stage.trigger" class="trigger">{{ formatTrigger(stage.trigger) }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Formas Disponíveis -->
        <div v-if="availableForms.length > 0" class="available-forms-section">
          <h3>Formas Disponíveis / Available Forms</h3>
          <div class="forms-grid">
            <div 
              v-for="form in availableForms" 
              :key="form.name"
              class="form-card"
              @click="loadPokemonByName(form.name)"
              :class="{ 'current-form': currentVariant === form.name }"
            >
              <img :src="getFormImage(form)" :alt="form.name">
              <span class="form-name">{{ getFormDisplayName(form) }}</span>
              <span class="form-type">{{ form.type }}</span>
            </div>
          </div>
        </div>

        <!-- Estatísticas -->
        <div v-if="pokemon.stats" class="stats-section">
          <h3>Estatísticas / Stats</h3>
          <div class="stats">
            <div v-for="stat in pokemon.stats" :key="stat.stat.name" class="stat">
              <div class="stat-name">{{ formatName(stat.stat.name) }}</div>
              <div class="stat-bar">
                <div class="stat-value" :style="{ width: (stat.base_stat / 255 * 100) + '%' }">
                  <span class="stat-number">{{ stat.base_stat }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

      </div>

      <div v-else class="not-found">
        <h2>Pokémon não encontrado / Pokémon not found</h2>
        <p>Tente buscar outro Pokémon / Try searching for another Pokémon</p>
      </div>

      <button class="close-bottom" @click="closeDetail">
        <i class="fas fa-times"></i> Fechar / Close
      </button>

    </div>
    <div v-else class="loading">
      <i class="fas fa-spinner fa-spin"></i>
      <p>Carregando Pokémon...</p>
    </div>
  </div>
</template>

<script>
export default {
  props: [
    'pokemonUrl',
    'imageUrl'
  ],
  data: () => {
    return {
      show: false,
      pokemon: null,
      speciesData: null,
      allForms: [],
      evolutionChain: [],
      currentVariant: null,
      variants: [],
      availableForms: [],
      loading: true
    }      
  },
  computed: {
    hasVariants() {
      return this.variants.length > 1;
    }
  },
  methods: {
    formatName(name) {
      if (!name) return '';
      return name.split('-')
        .map(word => word.charAt(0).toUpperCase() + word.slice(1))
        .join(' ');
    },
    
    formatTrigger(trigger) {
      const triggers = {
        'level-up': 'Level Up',
        'trade': 'Trade',
        'use-item': 'Use Item',
        'shed': 'Shed'
      };
      return triggers[trigger] || trigger;
    },
    
    async fetchData() {
      this.loading = true;
      this.pokemon = null;
      this.evolutionChain = [];
      this.variants = [];
      this.availableForms = [];
      
      try {
        // Primeiro, buscar o Pokémon principal
        const response = await fetch(this.pokemonUrl);
        if (!response.ok) throw new Error('Pokémon not found');
        
        this.pokemon = await response.json();
        this.currentVariant = this.pokemon.name;
        
        // Buscar dados adicionais em paralelo
        await Promise.all([
          this.fetchSpeciesData(),
          this.fetchAllForms()
        ]);
        
        // Preparar variantes
        this.prepareVariants();
        
        this.show = true;
      } catch (error) {
        console.error('Erro ao buscar Pokémon:', error);
        this.pokemon = null;
      } finally {
        this.loading = false;
      }
    },
    
    async fetchSpeciesData() {
      if (!this.pokemon?.species?.url) return;
      
      try {
        const response = await fetch(this.pokemon.species.url);
        this.speciesData = await response.json();
        
        // Buscar cadeia evolutiva
        if (this.speciesData.evolution_chain?.url) {
          await this.fetchEvolutionChain(this.speciesData.evolution_chain.url);
        }
      } catch (error) {
        console.error('Erro ao buscar dados da espécie:', error);
      }
    },
    
    async fetchAllForms() {
      if (!this.speciesData?.varieties) return;
      
      try {
        // Buscar dados de todas as variedades
        const formPromises = this.speciesData.varieties.map(variety => 
          fetch(variety.pokemon.url).then(res => res.json())
        );
        
        this.allForms = await Promise.all(formPromises);
      } catch (error) {
        console.error('Erro ao buscar formas:', error);
        this.allForms = [];
      }
    },
    
    async fetchEvolutionChain(url) {
      try {
        const response = await fetch(url);
        const chainData = await response.json();
        this.evolutionChain = this.processEvolutionChain(chainData.chain);
      } catch (error) {
        console.error('Erro ao buscar cadeia evolutiva:', error);
        this.evolutionChain = [];
      }
    },
    
    processEvolutionChain(chain, result = []) {
      const id = parseInt(chain.species.url.split('/').filter(Boolean).pop());
      
      result.push({
        id: id,
        name: chain.species.name,
        trigger: chain.evolves_to[0]?.evolution_details[0]?.trigger?.name || null
      });
      
      if (chain.evolves_to.length > 0) {
        chain.evolves_to.forEach(evolution => {
          this.processEvolutionChain(evolution, result);
        });
      }
      
      return result;
    },
    
    prepareVariants() {
      // Adicionar a forma atual como primeira variante
      const baseForm = {
        name: this.pokemon.name,
        displayName: 'Normal',
        type: 'Normal',
        sprite: this.pokemon.sprites?.front_default || `${this.imageUrl}${this.pokemon.id}.png`
      };
      
      this.variants = [baseForm];
      this.availableForms = [baseForm];
      
      // Analisar todas as formas disponíveis
      this.allForms.forEach(form => {
        // Pular a forma atual (já adicionada)
        if (form.name === this.pokemon.name) return;
        
        const formData = this.analyzeForm(form);
        
        if (formData) {
          this.variants.push(formData);
          this.availableForms.push(formData);
        }
      });
    },
    
    analyzeForm(form) {
      if (!form || !form.name || !this.pokemon) return null;
      
      const formName = form.name.toLowerCase();
      let displayName = 'Normal';
      let type = 'Normal';
      
      // Detectar tipo de forma
      if (formName.includes('mega')) {
        if (formName.includes('mega-x')) {
          displayName = 'Mega X';
          type = 'Mega X';
        } else if (formName.includes('mega-y')) {
          displayName = 'Mega Y';
          type = 'Mega Y';
        } else {
          displayName = 'Mega';
          type = 'Mega';
        }
      } else if (formName.includes('gmax') || formName.includes('gigantamax')) {
        displayName = 'Gigantamax';
        type = 'Gigantamax';
      } else if (formName.includes('alola')) {
        displayName = 'Alolan';
        type = 'Alola';
      } else if (formName.includes('galar')) {
        displayName = 'Galarian';
        type = 'Galar';
      } else if (formName.includes('hisui')) {
        displayName = 'Hisuian';
        type = 'Hisui';
      } else if (formName.includes('paldea')) {
        displayName = 'Paldean';
        type = 'Paldea';
      } else {
        // Para outras formas, usar o nome formatado
        displayName = this.getFormDisplayNameFromString(form.name);
        type = 'Alternative';
      }
      
      return {
        name: form.name,
        displayName: displayName,
        type: type,
        sprite: form.sprites?.front_default || `${this.imageUrl}${form.id}.png`,
        formData: form
      };
    },
    
    getFormDisplayNameFromString(formName) {
      if (!formName || !this.pokemon?.name) return 'Standard';
      
      const baseName = this.pokemon.name.split('-')[0];
      let displayName = formName.replace(new RegExp(`${baseName}-?`, 'i'), '');
      
      if (!displayName) return 'Standard';
      
      // Formata as palavras
      displayName = displayName.split('-')
        .map(word => {
          // Trata casos especiais
          if (word.toLowerCase() === 'gmax') return 'G-Max';
          if (word.toLowerCase() === 'mega') return 'Mega';
          return word.charAt(0).toUpperCase() + word.slice(1);
        })
        .filter(word => word)
        .join(' ');
      
      return displayName;
    },
    
    getCurrentImage() {
      if (this.currentVariant === this.pokemon.name) {
        return this.imageUrl + this.pokemon.id + '.png';
      }
      
      // Encontrar a variante atual
      const variant = this.variants.find(v => v.name === this.currentVariant);
      return variant ? variant.sprite : this.imageUrl + this.pokemon.id + '.png';
    },
    
    selectVariant(variant) {
      this.currentVariant = variant.name;
      
      // Se a variante for diferente do Pokémon atual, carregar seus dados
      if (variant.name !== this.pokemon.name) {
        this.loadPokemonByName(variant.name);
      }
    },
    
    getFormImage(form) {
      return form.sprite || `${this.imageUrl}${this.pokemon.id}.png`;
    },
    
    getFormDisplayName(form) {
      if (!form || !form.name || !this.pokemon?.name) return 'Standard';
      
      const baseName = this.pokemon.name.split('-')[0];
      let displayName = form.name.replace(new RegExp(`${baseName}-?`, 'i'), '');
      
      if (!displayName) return 'Standard';
      
      // Formata as palavras
      displayName = displayName.split('-')
        .map(word => {
          // Trata casos especiais
          if (word.toLowerCase() === 'gmax') return 'G-Max';
          if (word.toLowerCase() === 'mega') return 'Mega';
          return word.charAt(0).toUpperCase() + word.slice(1);
        })
        .filter(word => word)
        .join(' ');
      
      return displayName;
    },
    
    loadPokemonById(id) {
      this.$emit('selectPokemon', `https://pokeapi.co/api/v2/pokemon/${id}/`);
    },
    
    loadPokemonByName(name) {
      this.$emit('selectPokemon', `https://pokeapi.co/api/v2/pokemon/${name}/`);
    },
    
    closeDetail() {
      this.$emit('closeDetail');
    }
  },
  watch: {
    pokemonUrl: {
      immediate: true,
      handler() {
        if (this.pokemonUrl) {
          this.fetchData();
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped src="./index.scss" />