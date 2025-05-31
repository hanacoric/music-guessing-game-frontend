<template>
  <div class="app-wrapper">
    <StartGame v-if="!gameStarted && !showFinal" @started="handleGameStart" />

    <GuessSong
      v-if="gameStarted && genre && deezerGenreId && gameSessionId !== null"
      :genre="genre"
      :deezerGenreId="deezerGenreId"
      :gameSessionId="gameSessionId"
      :snippetDuration="snippetDuration"
      @gameFinished="handleGameFinished"
      @goToStart="resetToStart"
    />

    <FinalScore
      v-if="showFinal && gameSessionId !== null"
      :score="finalScore"
      :rounds="maxRounds"
      :gameSessionId="gameSessionId"
      @restart="resetToStart"
    />
  </div>
</template>

<script lang="ts">
import StartGame from './components/StartGame.vue'
import GuessSong from './components/GuessSong.vue'
import FinalScore from './components/FinalScore.vue'

export default {
  components: { StartGame, GuessSong, FinalScore },
  data() {
    return {
      genre: null as string | null,
      deezerGenreId: null as string | number | null,
      gameSessionId: null as string | number | null,
      snippetDuration: 5,
      gameStarted: false,
      showFinal: false,
      finalScore: 0,
      maxRounds: 5,
    }
  },
  methods: {
    handleGameStart({
      genre,
      deezerGenreId,
      gameSessionId,
      snippetDuration,
    }: {
      genre: string
      deezerGenreId: string | number
      gameSessionId: string | number
      snippetDuration: number
    }) {
      this.genre = genre
      this.deezerGenreId = deezerGenreId
      this.gameSessionId = gameSessionId
      this.snippetDuration = snippetDuration
      this.gameStarted = true
    },
    async handleGameFinished() {
      try {
        const res = await fetch(
          `${import.meta.env.VITE_BACKEND_URL}/api/game/score/${this.gameSessionId}`,
        )
        const data = await res.json()
        this.finalScore = data.score
      } catch (err) {
        console.error('Failed to fetch final score', err)
        this.finalScore = 0
      }

      this.showFinal = true
      this.gameStarted = false
    },

    resetToStart() {
      this.genre = null
      this.deezerGenreId = null
      this.gameSessionId = null
      this.finalScore = 0
      this.showFinal = false
      this.gameStarted = false
    },
  },
}
</script>

<style>
#app {
  display: flex !important;
  flex-direction: column;
  min-height: 100vh;
}

html,
body {
  height: 100%;
  margin: 0;
}

.app-wrapper {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Center FinalScore specifically */
.app-wrapper:has(> .final-score-box) {
  align-items: center;
  justify-content: center;
}
</style>
