<template>
  <!-- Contenedor principal de la aplicación -->
  <div class="app" @keydown="handleKeydown" tabindex="0">
    <!-- Mensaje de turno -->
    <div class="app__turn-message">
      <h2>{{ turnMessage }}</h2>
    </div>
    
    <!-- Tablero de juego -->
    <Board :board="board" :players="players" @cellClicked="handleCellClick" />

    <!-- Controles de la aplicación -->
    <div class="app__controls controls">
      <!-- Botón para cambiar entre modo de acción (move/wall) -->
      <button @click="toggleActionMode" :class="['controls__toggle-action-mode', { 'controls__toggle-action-mode--move': actionMode === 'move', 'controls__toggle-action-mode--wall': actionMode === 'wall' }]">
        {{ actionMode === 'move' ? 'Modo Bloqueo' : 'Modo Movimiento' }}
      </button>
      
      <!-- Controles específicos para modo wall -->
      <div v-if="actionMode === 'wall'" class="controls__wall wall-controls">
        <button @click="setWallOrientation('horizontal')" class="wall-controls__button">Horizontal</button>
        <button @click="setWallOrientation('vertical')" class="wall-controls__button">Vertical</button>
      </div>
      
      <!-- Controles específicos para modo move -->
      <div v-if="actionMode === 'move'" class="controls__move move-controls">
        <button @click="handleMove('up')" class="move-controls__button">Arriba</button>
        <button @click="handleMove('down')" class="move-controls__button">Abajo</button>
        <button @click="handleMove('left')" class="move-controls__button">Izquierda</button>
        <button @click="handleMove('right')" class="move-controls__button">Derecha</button>
      </div>
    </div>
  </div>
</template>

<script>
import Board from './components/organismos/Board.vue';

export default {
  components: { Board },
  data() {
    return {
      // Datos iniciales del juego
      board: this.initializeBoard(),
      players: [
        { id: 1, row: 0, col: 4, walls: 10, goalRow: 8 },
        { id: 2, row: 8, col: 4, walls: 10, goalRow: 0 },
      ],
      currentPlayer: 0,
      winner: null,
      turnMessage: 'Turno del jugador 1',
      actionMode: 'move',  // 'move' o 'wall'
      wallOrientation: 'horizontal',
    };
  },
  computed: {
    currentPlayerWalls() {
      return this.players[this.currentPlayer].walls;
    },
  },
  mounted() {
    this.$el.focus();
  },
  
  methods: {
    // Inicializa el tablero de juego
    initializeBoard() {
      const board = [];
      for (let i = 0; i < 9; i++) {
        const row = [];
        for (let j = 0; j < 9; j++) {
          row.push({ row: i, col: j, hWall: false, vWall: false });
        }
        board.push(row);
      }
      return board;
    },

    // Maneja el movimiento de los jugadores
    handleMove(direction) {
      if (this.actionMode !== 'move') return;

      const player = this.players[this.currentPlayer];
      let newRow = player.row;
      let newCol = player.col;

      switch (direction) {
        case 'up':
          newRow -= 1;
          break;
        case 'down':
          newRow += 1;
          break;
        case 'left':
          newCol -= 1;
          break;
        case 'right':
          newCol += 1;
          break;
      }

      if (this.isValidMove(player, newRow, newCol)) {
        player.row = newRow;
        player.col = newCol;
        if (this.checkWin(player)) {
          this.winner = player.id;
          this.turnMessage = `¡El jugador ${player.id} ha ganado!`;
        } else {
          this.switchPlayer();
        }
      } else {
        alert('Movimiento inválido');
      }
    },

    // Cambia el modo de acción entre move y wall
    toggleActionMode() {
      this.actionMode = this.actionMode === 'move' ? 'wall' : 'move';
    },

    // Establece la orientación del muro (horizontal/vertical)
    setWallOrientation(orientation) {
      this.wallOrientation = orientation;
    },

    // Valida si un movimiento es válido
    isValidMove(player, row, col) {
      const rowDiff = Math.abs(player.row - row);
      const colDiff = Math.abs(player.col - col);
      if (rowDiff + colDiff !== 1) return false;

      for (const otherPlayer of this.players) {
        if (otherPlayer !== player && otherPlayer.row === row && otherPlayer.col === col) {
          return false;
        }
      }

      if (rowDiff === 1) {
        if (player.row < row && this.board[player.row][player.col].hWall) return false;
        if (player.row > row && this.board[row][col].hWall) return false;
      }
      if (colDiff === 1) {
        if (player.col < col && this.board[player.row][player.col].vWall) return false;
        if (player.col > col && this.board[row][col].vWall) return false;
      }

      return row >= 0 && row < 9 && col >= 0 && col < 9;
    },

    // Maneja el evento de tecla presionada
    handleKeydown(event) {
      const keyMap = {
        ArrowUp: 'up',
        ArrowDown: 'down',
        ArrowLeft: 'left',
        ArrowRight: 'right',
      };
      const direction = keyMap[event.key];
      if (direction && this.actionMode === 'move') {
        this.handleMove(direction);
      }
    },

    // Verifica si un jugador ha ganado
    checkWin(player) {
      return player.row === player.goalRow;
    },

    // Cambia al siguiente jugador
    switchPlayer() {
      this.currentPlayer = this.currentPlayer === 0 ? 1 : 0;
      this.turnMessage = `Turno del jugador ${this.currentPlayer + 1}`;
      this.actionMode = 'move';
    },

    // Valida si es posible colocar un muro en una posición dada
    isValidWallPlacement(row, col, orientation) {
      if (orientation === 'horizontal') {
        if (
          col >= 8 ||
          this.board[row][col].hWall || this.board[row][col + 1].hWall
        ) {
          return false;
        }
        if (
          (row > 0 && this.board[row - 1][col].vWall && this.board[row - 1][col + 1].vWall) ||
          (row < 8 && this.board[row + 1][col].vWall && this.board[row + 1][col + 1].vWall)
        ) {
          return false;
        }
      } else {
        if (
          row >= 8 ||
          this.board[row][col].vWall || this.board[row + 1][col].vWall
        ) {
          return false;
        }
        if (
          (col > 0 && this.board[row][col - 1].hWall && this.board[row + 1][col - 1].hWall) ||
          (col < 8 && this.board[row][col + 1].hWall && this.board[row + 1][col + 1].hWall)
        ) {
          return false;
        }
      }
      return true;
    },

    // Verifica si hay un camino disponible para un jugador
    isPathAvailable(player) {
      const directions = [
        { row: -1, col: 0 },
        { row: 1, col: 0 },
        { row: 0, col: -1 },
        { row: 0, col: 1 }
      ];

      const visited = Array.from({ length: 9 }, () => Array(9).fill(false));
      const queue = [{ row: player.row, col: player.col }];
      visited[player.row][player.col] = true;

      while (queue.length > 0) {
        const { row, col } = queue.shift();

        if (row === player.goalRow) return true;

        for (const direction of directions) {
          const newRow = row + direction.row;
          const newCol = col + direction.col;

          if (
            newRow >= 0 && newRow < 9 &&
            newCol >= 0 && newCol < 9 &&
            !visited[newRow][newCol] &&
            this.isValidMove({ row, col }, newRow, newCol)
          ) {
            visited[newRow][newCol] = true;
            queue.push({ row: newRow, col: newCol });
          }
        }
      }

      return false;
    },

    // Maneja la colocación de un muro en una celda
    handlePlaceWall(row, col) {
      const player = this.players[this.currentPlayer];

      if (this.isValidWallPlacement(row, col, this.wallOrientation)) {
        if (this.wallOrientation === 'horizontal') {
          this.board[row][col].hWall = true;
          this.board[row][col + 1].hWall = true;
        } else {
          this.board[row][col].vWall = true;
          this.board[row + 1][col].vWall = true;
        }

        if (this.isPathAvailable(this.players[0]) && this.isPathAvailable(this.players[1])) {
          player.walls -= 1;
          this.switchPlayer();
        } else {
          // Si el muro bloquea el camino de algún jugador, se deshace la acción
          if (this.wallOrientation === 'horizontal') {
            this.board[row][col].hWall = false;
            this.board[row][col + 1].hWall = false;
          } else {
            this.board[row][col].vWall = false;
            this.board[row + 1][col].vWall = false;
          }
          alert('Colocación de muro inválida, bloquea el camino de un jugador');
        }
      }
    },

    // Maneja el click en una celda del tablero
    handleCellClick({ row, col }) {
      if (this.actionMode === 'wall') {
        this.handlePlaceWall(row, col);
      }
    },
  },
};
</script>

<style scoped>
/* Estilos del componente App */
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.app__turn-message {
  margin-bottom: 10px;
}

.app__controls {
  margin-top: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* Estilos de los controles */
.controls__toggle-action-mode {
  background-color: #f0f0f0;
  border: 1px solid #ccc;
  padding: 5px 10px;
  cursor: pointer;
}

.controls__toggle-action-mode--move {
  background-color: #4caf50;
  color: white;
}

.controls__toggle-action-mode--wall {
  background-color: #f44336;
  color: white;
}

.controls__wall {
  display: flex;
  gap: 10px;
}

.controls__move {
  display: flex;
  gap: 10px;
}

/* Estilos de los botones */
.wall-controls__button,
.move-controls__button {
  background-color: #f0f0f0;
  border: 1px solid #ccc;
  padding: 5px 10px;
  cursor: pointer;
}

</style>