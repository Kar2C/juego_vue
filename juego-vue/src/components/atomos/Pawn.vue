<template>
  <!-- Contenedor principal con clases dinámicas -->
  <div :class="['pawn', 'pawn--player' + player.id]"></div>
</template>

<script>
export default {
  props: {
    // Propiedad que recibe el objeto jugador
    player: Object,
  }
};
</script>

<style scoped>
/* Estilos del componente Pawn */

/* Estilo base para el peón */
.pawn {
  width: 100%;
  height: 100%;
  border-radius: 50%;
}

/* Modificador BEM para el peón del jugador 1 */
.pawn--player1 {
  background-color: red;
}

/* Modificador BEM para el peón del jugador 2 */
.pawn--player2 {
  background-color: blue;
}
</style>
