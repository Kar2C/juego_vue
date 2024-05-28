<template>
  <!-- Contenedor principal con clases y estilos BEM -->
  <div class="row">
    <Cell v-for="cell in row" :key="cell.col" :cell="cell" :players="players" @click="handleClick(cell)" />
  </div>
</template>

<script>
import Cell from '../atomos/Cell.vue';

export default {
  components: { Cell },
  props: {
    // Propiedades del componente
    row: Array,
    players: Array,
  },
  methods: {
    // Método para manejar el clic en una celda
    handleClick(cell) {
      this.$emit('cellClicked', cell);
    }
  }
};
</script>

<style scoped>
/* Estilos del componente Row */

/* Contenedor principal del componente Row */
.row {
  display: flex;
}
</style>
