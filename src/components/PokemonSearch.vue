<template>
  <div class="searchbar">
    <form @submit.prevent="setPokemonUrl">
      <input type="text" placeholder="Número na Pokedex / Number on Pokedex" v-model="searchvalue">
    </form>
    <i class="fas fa-search" @click="setPokemonUrl"></i>
  </div>
</template>

<script>
  export default {
    props: [
      'apiUrl'
    ],
    data: () => {
      return {
        searchvalue: '',
      }
    },
    methods: {
      setPokemonUrl() {
        if(this.searchvalue !== '')
          this.$emit('setPokemonUrl', this.apiUrl + this.searchvalue);
      }
    }
  }
</script>

<style lang="scss" scoped>
    .searchbar {
      max-width: 510px;
      padding-bottom: 20px;
      position: relative;
      width: 100%;
    }

    input {
      border: none;
      border-radius: 5px;
      box-shadow: 0 15px 30px rgba(0,0,0,.2), 0 10px 10px rgba(0,0,0,.2);
      font-size: 1rem;
      outline: none;
      padding: 10px 40px 10px 10px;
      width: calc(100% - 50px);
    }
    
    i {
      color: #0A2E50;
      cursor: pointer;
      font-size: 1.25rem;
      position: absolute;
      right: 10px;
      top: 10px;
    }
</style>
