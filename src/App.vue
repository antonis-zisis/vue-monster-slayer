<template>
  <header>
    <h1>Monster Slayer</h1>
  </header>

  <div>
    <section id="monster" class="container">
      <h2>Monster Health</h2>
      <div class="healthbar">
        <div class="healthbar__value" :style="monsterBarStyle"></div>
      </div>
    </section>

    <section id="player" class="container">
      <h2>Your Health</h2>
      <div class="healthbar">
        <div class="healthbar__value" :style="playerBarStyle"></div>
      </div>
    </section>

    <section class="container" v-if="winner">
      <h2>Game Over!</h2>
      <h3 v-if="winner == 'monster'">You lost!</h3>
      <h3 v-else-if="winner == 'player'">You won!</h3>
      <h3 v-else>It's a draw!</h3>
      <button @click="startGame">START GAME</button>
    </section>

    <section id="controls" v-else>
      <button @click="attackMonster">ATTACK</button>
      <button @click="specialAttack" :disabled="canUseSpecialAttack">
        SPECIAL ATTACK
      </button>
      <button @click="healPlayer">HEAL</button>
      <button @click="surrender">SURRENDER</button>
    </section>

    <section id="log" class="container">
      <h2>Battle Log</h2>
      <ul>
        <li v-for="log in logs" :key="log">
          <span
            :class="{
              'log--player': log.actor === 'player',
              'log--monster': log.actor === 'monster',
            }"
            >{{ log.actor === 'player' ? 'Player' : 'Monster' }}</span
          >

          <span v-if="log.action === 'heal'">
            heals himself for
            <span class="log--heal">{{ log.value }}</span>
          </span>

          <span v-else>
            attacks and deals
            <span class="log--damage">{{ log.value }} damage</span>
          </span>
        </li>
      </ul>
    </section>
  </div>
</template>

<script>
function getRandomValue(min, max) {
  return Math.floor(Math.random() * (max - min)) + min;
}

export default {
  name: 'App',
  data() {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      currentRound: 0,
      winner: null,
      logs: [],
    };
  },
  computed: {
    monsterBarStyle() {
      if (this.monsterHealth < 0) {
        return { width: '0%' };
      }

      return { width: `${this.monsterHealth}%` };
    },
    playerBarStyle() {
      if (this.playerHealth < 0) {
        return { width: '0%' };
      }

      return { width: `${this.playerHealth}%` };
    },
    canUseSpecialAttack() {
      return this.currentRound % 3 !== 0;
    },
  },
  watch: {
    playerHealth(value) {
      if (value <= 0 && this.monsterHealth <= 0) {
        this.winner = 'draw';
      } else if (value <= 0) {
        this.winner = 'monster';
      }
    },
    monsterHealth(value) {
      if (value <= 0 && this.playerHealth <= 0) {
        this.winner = 'draw';
      } else if (value <= 0) {
        this.winner = 'player';
      }
    },
  },
  methods: {
    startGame() {
      this.playerHealth = 100;
      this.monsterHealth = 100;
      this.currentRound = 0;
      this.winner = null;
      this.logs = [];
    },
    attackMonster() {
      const attackValue = getRandomValue(5, 15);
      this.monsterHealth = this.monsterHealth - attackValue;
      this.attackPlayer();
      this.logger('player', 'attack', attackValue);
      this.increaseCurrentRound();
    },
    attackPlayer() {
      const attackValue = getRandomValue(10, 20);
      this.playerHealth = this.playerHealth - attackValue;
      this.logger('monster', 'attack', attackValue);
    },
    specialAttack() {
      const attackValue = getRandomValue(15, 25);
      this.monsterHealth = this.monsterHealth - attackValue;
      this.attackPlayer();
      this.increaseCurrentRound();
      this.logger('player', 'attack', attackValue);
    },
    healPlayer() {
      const healValue = getRandomValue(15, 20);

      if (this.playerHealth + healValue > 100) {
        this.playerHealth = 100;
      } else {
        this.playerHealth = this.playerHealth + healValue;
      }

      this.attackPlayer();
      this.increaseCurrentRound();
      this.logger('player', 'heal', healValue);
    },
    surrender() {
      this.winner = 'monster';
    },
    increaseCurrentRound() {
      this.currentRound = this.currentRound + 1;
    },
    logger(actor, action, value) {
      this.logs.unshift({ actor, action, value });
    },
  },
};
</script>

<style>
* {
  box-sizing: border-box;
}

html {
  font-family: 'Jost', sans-serif;
}

body {
  margin: 0;
}

header {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  padding: 0.5rem;
  background-color: #880017;
  color: white;
  text-align: center;
  margin-bottom: 2rem;
}

section {
  width: 90%;
  max-width: 40rem;
  margin: auto;
}

.healthbar {
  width: 100%;
  height: 40px;
  border: 1px solid #575757;
  margin: 1rem 0;
  background: #fde5e5;
}

.healthbar__value {
  background-color: #00a876;
  width: 100%;
  height: 100%;
}

.container {
  text-align: center;
  padding: 0.5rem;
  margin: 1rem auto;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  border-radius: 12px;
}

#monster h2,
#player h2 {
  margin: 0.25rem;
}

#controls {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

button {
  font: inherit;
  border: 1px solid #88005b;
  background-color: #88005b;
  color: white;
  padding: 1rem 2rem;
  border-radius: 12px;
  margin: 1rem;
  width: 14rem;
  cursor: pointer;
  box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.26);
}

button:focus {
  outline: none;
}

button:hover,
button:active {
  background-color: #af0a78;
  border-color: #af0a78;
  box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.26);
}

button:disabled {
  background-color: #ccc;
  border-color: #ccc;
  box-shadow: none;
  color: #3f3f3f;
  cursor: not-allowed;
}

#log ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#log li {
  margin: 0.5rem 0;
}

.log--player {
  color: #7700ff;
}

.log--monster {
  color: #da8d00;
}

.log--damage {
  color: red;
}

.log--heal {
  color: green;
}
</style>
