<template>
  <!-- Contenedor principal con clases y estilos BEM -->
  <div class="board">
    <!-- Itera sobre las filas del tablero -->
    <Row v-for="(row, rowIndex) in board" :key="rowIndex" :row="row" :players="players" @cellClicked="handleCellClick" />
    <!-- genera las celdad del tablero  -->
  </div>
</template>
<script>
// Importamos el componente Row desde su ubicación
import Row from '../moleculas/Row.vue';

export default {
  // Declaramos los componentes que utiliza este componente
  components: { Row },

  // Declaramos las propiedades que este componente espera recibir
  props: {
    board: Array,     // Arreglo que representa el tablero
    players: Array,   // Arreglo que representa a los jugadores
  },

  // Métodos disponibles en este componente
  methods: {
    // Método para manejar el clic en una celda
    handleCellClick(cell) {
      this.$emit('cellClicked', cell);  // Emitimos el evento 'cellClicked' con la celda como argumento
    }
  }
};
</script>
<style scoped>
/* Estilos del componente Board */

/* Contenedor principal del componente Board */
.board {
  display: flex;            /* Se muestra como un contenedor flexible */
  flex-direction: column;   /* Dirección de los elementos: columna */
}
</style>
