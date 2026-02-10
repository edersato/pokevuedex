<template>
  <div class="list">
      <article 
        v-for="(pokemon, index) in displayedPokemons" 
        :key="'poke'+index" 
        @click="setPokemonUrl(pokemon.url)"
        :class="{ 'inactive': !isInSelectedRegion(pokemon.id) }"
      >
        <img :src="getPokemonImage(pokemon)" class="pokemonSprite" alt="">
        <h3>{{ pokemon.id }} - {{ formatName(pokemon.name) }}</h3>
        <div v-if="!isInSelectedRegion(pokemon.id)" class="region-warning">
          ⚠️ Fora da região selecionada
        </div>
      </article>

      <!-- Scroll infinito apenas para "Todas" -->
      <div v-if="hasMorePokemons && selectedRegionId === 10" id="scroll-trigger" ref="infinitescrolltrigger">
          <i class="fas fa-spinner fa-spin"></i>
          <p>Carregando mais Pokémon...</p>
      </div>
      
      <!-- Mensagem para quando não há Pokémon na região -->
      <div v-if="selectedRegionId !== 10 && displayedPokemons.length === 0 && !loading" class="no-pokemons">
        <i class="fas fa-search"></i>
        <p>Nenhum Pokémon encontrado nesta região</p>
        <button @click="loadAllPokemons" class="load-all-btn">
          Carregar todos os Pokémon
        </button>
      </div>
      
      <!-- Loading para regiões específicas -->
      <div v-if="selectedRegionId !== 10 && loading" class="loading-region">
        <i class="fas fa-spinner fa-spin"></i>
        <p>Carregando Pokémon da região {{ selectedRegionName }}...</p>
      </div>
  </div>
</template>

<script>
import { regions } from "../../data/region.js"

export default {
    props: [
        'imageUrl',
        'apiUrl',
        'region'
    ],
    data: () => {
        return {
            pokemons: [],
            nextUrl: '',
            currentUrl: '',
            regions: regions,
            loading: false,
            allPokemonsLoaded: false,
            loadingRegion: false
        }
    },
    computed: {
        selectedRegion() {
            return this.regions.find(r => r.id === this.region) || this.regions[this.regions.length - 1];
        },
        selectedRegionId() {
            return this.selectedRegion.id;
        },
        selectedRegionName() {
            return this.selectedRegion.name;
        },
        displayedPokemons() {
            // Se for "Todas", mostra tudo
            if (this.selectedRegionId === 10) {
                return this.pokemons;
            }
            
            // Para regiões específicas, mostra apenas Pokémon dentro do intervalo
            // Mas mantém os outros carregados para mudança rápida de região
            return this.pokemons.filter(pokemon => 
                this.isInSelectedRegion(pokemon.id)
            );
        },
        hasMorePokemons() {
            return this.nextUrl && !this.allPokemonsLoaded && this.selectedRegionId === 10;
        }
    },
    methods: {
        getPokemonImage(pokemon) {
            return `${this.imageUrl}${pokemon.id}.png`;
        },
        
        formatName(name) {
            if (!name) return 'Loading...';
            return name.split('-')
                .map(word => word.charAt(0).toUpperCase() + word.slice(1))
                .join(' ');
        },
        
        isInSelectedRegion(pokemonId) {
            if (this.selectedRegionId === 10) return true;
            return pokemonId >= this.selectedRegion.start && 
                   pokemonId <= this.selectedRegion.end;
        },
        
        fetchData() {
            if (this.loading || this.allPokemonsLoaded) return;
            
            this.loading = true;
            let req = new Request(this.currentUrl);
            
            fetch(req)
            .then((resp) => {
                if(resp.status === 200)
                    return resp.json();
                throw new Error('Failed to fetch');
            })
            .then((data) => {
                this.nextUrl = data.next;
                
                // Se não há mais resultados, marcar como carregado
                if (!data.next) {
                    this.allPokemonsLoaded = true;
                }
                
                // Processar cada Pokémon
                data.results.forEach(pokemon => {
                    pokemon.id = parseInt(pokemon.url.split('/').filter(part => !!part).pop());
                    this.pokemons.push(pokemon);
                });
                
                this.loading = false;
                
                // Se estamos carregando uma região específica e ainda não temos todos
                if (this.selectedRegionId !== 9) {
                    this.ensureRegionPokemonsLoaded();
                }
            })
            .catch((error) => {
                console.error('Erro ao buscar Pokémon:', error);
                this.loading = false;
            });
        },
        
        // Garante que todos Pokémon da região selecionada estejam carregados
        ensureRegionPokemonsLoaded() {
            if (this.selectedRegionId === 9) return;
            
            const regionPokemons = this.pokemons.filter(p => 
                this.isInSelectedRegion(p.id)
            );
            
            // Se não temos todos Pokémon da região e ainda podemos carregar mais
            if (regionPokemons.length < (this.selectedRegion.end - this.selectedRegion.start + 1) && 
                this.nextUrl && !this.loading) {
                this.next();
            }
        },
        
        scrollTrigger() {
            this.$nextTick(() => {
                if (!this.$refs.infinitescrolltrigger) return;
                
                const observer = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if(entry.intersectionRatio > 0 && this.nextUrl && !this.loading && this.selectedRegionId === 9) {
                            this.next();
                        }
                    });
                }, {
                    threshold: 0.1
                });

                observer.observe(this.$refs.infinitescrolltrigger);
            });
        },
        
        next() {
            this.currentUrl = this.nextUrl;
            this.fetchData();
        },
        
        setPokemonUrl(url) {
            this.$emit('setPokemonUrl', url);
        },
        
        // Carrega Pokémon específicos da região
        loadRegionPokemons() {
            if (this.selectedRegionId === 10) {
                return; // Para "Todas", já está carregando via scroll infinito
            }
            
            this.loadingRegion = true;
            const start = this.selectedRegion.start;
            const end = this.selectedRegion.end;
            
            // Verificar quais Pokémon já temos carregados
            const missingIds = [];
            for (let id = start; id <= end; id++) {
                if (!this.pokemons.find(p => p.id === id)) {
                    missingIds.push(id);
                }
            }
            
            // Se já temos todos, apenas marcar como carregado
            if (missingIds.length === 0) {
                this.loadingRegion = false;
                return;
            }
            
            // Carregar Pokémon que faltam em lotes para não sobrecarregar
            this.loadPokemonsBatch(missingIds, 0);
        },
        
        async loadPokemonsBatch(ids, startIndex) {
            const batchSize = 11;
            const endIndex = Math.min(startIndex + batchSize, ids.length);
            
            const batchPromises = ids.slice(startIndex, endIndex).map(async (id) => {
                try {
                    const response = await fetch(`${this.apiUrl}${id}/`);
                    if (response.ok) {
                        const data = await response.json();
                        return {
                            id: data.id,
                            name: data.name,
                            url: `${this.apiUrl}${data.id}/`
                        };
                    }
                } catch (error) {
                    console.error(`Erro ao buscar Pokémon ${id}:`, error);
                }
                return null;
            });
            
            const batchResults = await Promise.all(batchPromises);
            batchResults.filter(p => p !== null).forEach(pokemon => {
                // Adicionar apenas se não existir
                if (!this.pokemons.find(p => p.id === pokemon.id)) {
                    this.pokemons.push(pokemon);
                }
            });
            
            // Se ainda há mais para carregar, continuar
            if (endIndex < ids.length) {
                setTimeout(() => {
                    this.loadPokemonsBatch(ids, endIndex);
                }, 500); // Pequeno delay para não sobrecarregar a API
            } else {
                this.loadingRegion = false;
            }
        },
        
        // Carrega todos Pokémon (para quando o usuário quiser ver todos)
        loadAllPokemons() {
            this.$emit('changeRegion', 10); // Muda para "Todas"
        },
        
        resetList() {
            // Não resetar completamente, apenas garantir que a região atual está visível
            if (this.selectedRegionId !== 10) {
                this.loadRegionPokemons();
            }
        }
    },
    watch: {
        region() {
            this.resetList();
        }
    },
    created() {
        this.currentUrl = this.apiUrl;
        this.fetchData();
    },
    mounted() {
        this.scrollTrigger();
    }
}
</script>

<style lang="scss" scoped src="./index.scss" />