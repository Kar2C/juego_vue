<template>
  <div class="cell" @click="handleClick">
    <!-- Renderiza el componente Pawn si hay un jugador en esta celda -->
    <Pawn v-if="hasPlayer" :player="player" />

    <!-- Renderiza muro horizontal si la propiedad hWall de la celda es verdadera -->
    <div v-if="cell.hWall" class="wall wall--horizontal"></div>

    <!-- Renderiza muro vertical si la propiedad vWall de la celda es verdadera -->
    <div v-if="cell.vWall" class="wall wall--vertical"></div>
  </div>
</template>

<script>
import Pawn from './Pawn.vue'; // Importa el componente Pawn

export default {
  components: { Pawn }, // Declara que el componente Pawn está siendo utilizado
  props: {
    cell: Object, // Propiedad que recibe el objeto celda
    players: Array, // Propiedad que recibe el array de jugadores
  },
  computed: {
    // Computed PROPIEDAD que verifica si hay un jugador en esta celda
    hasPlayer() { 
      return this.players.some(player => player.row === this.cell.row && player.col === this.cell.col);
      // almenos un jugador en una celda, true o false, 
    },
    // Computed property que obtiene el jugador en esta celda
    player() {
      return this.players.find(player => player.row === this.cell.row && player.col === this.cell.col);
      //pasa el primer resultado que encuentre
    }
  },
  methods: {
    // Método que se ejecuta cuando se hace click en la celda
    handleClick() {
      this.$emit('click', this.cell); // Emite un evento 'click' y pasa la celda como parámetro
    }
  }
};
</script>

<style scoped>
/* Estilos del componente Cell */

/* Estilo base para la celda */
.cell {
  width: 40px;
  height: 40px;
  border: 1px solid #000;
  position: relative;
}

/* Estilo base para los muros */
.wall {
  position: absolute;
  background-color: black;
}

/* Modificador BEM para muro horizontal */
.wall--horizontal {
  width: 100%;
  height: 5px;
  bottom: -5px; /* Coloca el muro 5px por debajo del borde inferior de la celda */
}

/* Modificador BEM para muro vertical */
.wall--vertical {
  width: 5px;
  height: 100%;
  right: -5px; /* Coloca el muro 5px a la derecha del borde derecho de la celda */
}
</style>
