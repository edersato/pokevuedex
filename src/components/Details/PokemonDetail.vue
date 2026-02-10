<template>
  <div class="detail" @click.self="closeDetail">
    <div class="detail-view card" v-if="show">
      <!-- Botão de fechar no topo -->
      <button class="close-btn" @click="closeDetail">
        <i class="fas fa-times"></i>
      </button>
      
      <div v-if="pokemon" class="image">
        <img :src="imageUrl + pokemon.id + '.png'" alt="">
      </div>

      <div v-if="pokemon" class="data card-body">
        <h2 class="card-title">{{ formatName(pokemon.name) }}</h2>
        
        <!-- Navegação entre variantes -->
        <div class="variants-nav" v-if="hasVariants">
          <button 
            v-for="variant in variants" 
            :key="variant.type"
            @click="selectVariant(variant)"
            :class="{ 'active': currentVariant === variant.type }"
            class="variant-btn"
          >
            {{ variant.label }}
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

        <!-- Variantes Regionais -->
        <div v-if="regionalVariants.length > 0" class="regional-variants-section">
          <h3>Variantes Regionais / Regional Variants</h3>
          <div class="variants-grid">
            <div 
              v-for="variant in regionalVariants" 
              :key="variant.name"
              class="variant-card"
              @click="loadPokemonByName(variant.name)"
            >
              <img :src="getVariantImage(variant.name)" :alt="variant.name">
              <span class="variant-name">{{ getFormDisplayName(variant.name) }}</span>
              <span class="variant-region">{{ variant.region }}</span>
            </div>
          </div>
        </div>

        <!-- Formas Especiais -->
        <div v-if="specialForms.length > 0" class="special-forms-section">
          <h3>Formas Especiais / Special Forms</h3>
          <div class="forms-grid">
            <div 
              v-for="form in specialForms" 
              :key="form.name"
              class="form-card"
              @click="loadPokemonByName(form.name)"
            >
              <img :src="getFormImage(form)" :alt="form.name">
              <span class="form-name">{{ getFormDisplayName(form.name) }}</span>
              <span v-if="form.type" class="form-type">{{ form.type }}</span>
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
      formsData: [],
      evolutionChain: [],
      currentVariant: 'normal',
      variants: [
        { type: 'normal', label: 'Normal' }
      ],
      regionalVariants: [],
      specialForms: [],
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
      this.regionalVariants = [];
      this.specialForms = [];
      
      try {
        const response = await fetch(this.pokemonUrl);
        if (!response.ok) throw new Error('Pokémon not found');
        
        this.pokemon = await response.json();
        
        // Buscar dados adicionais em paralelo
        await Promise.all([
          this.fetchSpeciesData(),
          this.fetchFormsData()
        ]);
        
        // Analisar variantes
        this.analyzeVariants();
        
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
    
    async fetchFormsData() {
      if (!this.pokemon?.forms) return;
      
      try {
        const formPromises = this.pokemon.forms
          .filter(form => form.url !== this.pokemonUrl) // Não incluir a forma atual
          .map(form => fetch(form.url).then(res => res.json()));
        
        this.formsData = await Promise.all(formPromises);
      } catch (error) {
        console.error('Erro ao buscar dados das formas:', error);
        this.formsData = [];
      }
    },
    
    analyzeVariants() {
      const baseName = this.pokemon.name.split('-')[0].toLowerCase();
      
      // Resetar variantes
      this.variants = [{ type: 'normal', label: 'Normal' }];
      this.regionalVariants = [];
      this.specialForms = [];
      
      // Analisar todas as formas
      this.formsData.forEach(form => {
        const formName = form.name.toLowerCase();
        
        // Mega Evolution
        if (formName.includes('mega')) {
          const megaType = formName.includes('mega-x') ? 'Mega X' : 
                          formName.includes('mega-y') ? 'Mega Y' : 'Mega';
          
          this.variants.push({
            type: formName,
            label: megaType
          });
          
          this.specialForms.push({
            name: form.name,
            type: megaType,
            sprite: form.sprites?.front_default
          });
        }
        
        // Gigantamax
        if (formName.includes('gmax') || formName.includes('gigantamax')) {
          this.variants.push({
            type: 'gmax',
            label: 'Gigantamax'
          });
          
          this.specialForms.push({
            name: form.name,
            type: 'Gigantamax',
            sprite: form.sprites?.front_default
          });
        }
        
        // Variantes Regionais
        const regions = ['alola', 'galar', 'hisui', 'paldea'];
        regions.forEach(region => {
          if (formName.includes(region)) {
            const regionName = region.charAt(0).toUpperCase() + region.slice(1);
            this.regionalVariants.push({
              name: form.name,
              region: regionName
            });
          }
        });
      });
    },
    
    selectVariant(variant) {
      this.currentVariant = variant.type;
      // Em uma implementação completa, aqui você buscaria os dados da variante
    },
    
    getVariantImage(name) {
      // Extrair ID do nome (assumindo formato "pokemon-region")
      const parts = name.split('-');
      const pokemonId = parts[0];
      
      // Para variantes, usar sprite específico se disponível
      return `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemonId}.png`;
    },
    
    getFormImage(form) {
      return form.sprite || `${this.imageUrl}${this.pokemon.id}.png`;
    },
    
    getFormDisplayName(name) {
      // Remove o nome base do Pokémon e formata
      const baseName = this.pokemon.name.split('-')[0];
      let displayName = name.replace(new RegExp(`${baseName}-?`, 'i'), '');
      
      // Formata as palavras
      displayName = displayName.split('-')
        .map(word => {
          // Trata casos especiais
          if (word.toLowerCase() === 'gmax') return 'G-Max';
          if (word.toLowerCase() === 'mega') return 'Mega';
          return word.charAt(0).toUpperCase() + word.slice(1);
        })
        .filter(word => word) // Remove strings vazias
        .join(' ');
      
      return displayName || 'Standard';
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