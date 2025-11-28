<template>
  <div class="my-4 text-center justify-center ">
    <h1 class="text-h4 font-weight-bold">PBJ FILMS (v2) </h1>
    <!-- <FlagIcon circle code="us" size="32" /> -->
  </div>
  <v-container fluid>
    <v-row align="center" class="fill-height" justify="center">
      <v-col class="d-flex flex-wrap" cols="12" justify="around">

        <v-card
          v-for="(movie, index) in movies"
          :key="movie.tmdb_id"
          align="center"
          class="mx-auto"
        >
          <v-img
            v-if="movie.poster_path"
            class="mx-auto pa-2 ma-2"
            cover
            height="450px"
            :src="movie.poster_path"
            width="300px"
          />

          <v-card-subtitle v-if="movie.director"><v-badge :content="movie.director" inline><v-icon icon="mdi-movie-open" /></v-badge>
          </v-card-subtitle>

          <v-badge v-if="movie.release_date" color="primary" :content="movie.release_date" inline>
            <v-icon icon="mdi-cake" />
          </v-badge>
          <v-badge color="warning" :content="`${movie.runtime} min`" inline><v-icon icon="mdi-clock" /></v-badge>
          <v-badge
            bordered
            color="success"
            :content="movie.date_watched"
            inline
            location="right center"
          >
            <v-icon icon="mdi-calendar" />
          </v-badge>

          <v-badge color="secondary" :content="movieNumber(index)" inline><v-icon icon="mdi-counter" /></v-badge>

          <v-card-actions class="justify-center">
            <v-btn class="text-amber" icon="mdi-account-group" @click="onShowCastClick(movie)" />
            <v-btn v-if="movie.oscars" class="text-teal" icon="mdi-trophy" @click="onShowOscarsClick(movie)" />

          </v-card-actions>

          <v-dialog v-model="movie.movieCastVisible" width="auto">
            <v-card prepend-icon="mdi-account-group" :subtitle="movie.director" :title="movie.title">
              <v-list-item
                v-for="person in movie.credits.cast.slice(0, 9)"
                :key="person.id"
                :subtitle="person.character"
                :title="person.name"
              />
            </v-card>
          </v-dialog>

          <v-dialog v-model="movie.oscarsVisible" width="auto">
            <v-card prepend-icon="mdi-trophy"><v-card-text><pre>{{ formatOscarResults(analyzeOscars(movie.oscars)) }}</pre></v-card-text></v-card>
          </v-dialog>

          <v-list-item />
        </v-card>
      </v-col>
    </v-row>
  </v-container>

</template>

<script setup>
  import FlagIcon from 'vue3-flag-icons'
  import { supabase } from '../utils/supabase'
  const movies = ref([])
  // const page = ref(1)
  // const itemsPerPage = ref(5)

  // const pageCount = computed(() => {
  //   return Math.ceil(movies.value.length / itemsPerPage.value)
  // })
  async function getMovies () {
    const { data } = await supabase.from('pbj-movies').select().order('date_watched', { ascending: false })
    movies.value = data

    for (const movie of movies.value) {
      getDirector(movie)
      getDetails(movie)
      getOscars(movie)
    }
  }
  function onShowCastClick (movie) {
    movie.movieCastVisible = !movie.movieCastVisible
  }

  function onShowOscarsClick (movie) {
    movie.oscarsVisible = !movie.oscarsVisible
  }

  function movieNumber (index) {
    return movies.value.length - index
  }
  async function getDetails (movie) {
    if (!movie.poster_path || !movie.release_date || !movie.runtime || !movie.genres) {
      const options = {
        method: 'GET',
        headers: {
          accept: 'application/json',
          Authorization: 'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJiNTY5Y2Y4ZTIyZTlmYTRiOWZkMGQzY2RhNzdlOGExZSIsIm5iZiI6MTc2MDcyMjE3Mi4wMjMsInN1YiI6IjY4ZjI3Y2ZjZjg4NzEyMjg3ZmVjNWMxZCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.GljyKUapeItmVRL7SgnNHWU0Z9WacUn2MN0rfQFcD7w',
        },
      }

      const response = await fetch(`https://api.themoviedb.org/3/movie/${movie.tmdb_id}?language=en-US`, options)
        .catch(error => console.error(error))

      if (!response) {
        throw new Error('Could not fetch resource.')
      }

      const data = await response.json()
      if (!movie.poster_path) {
        const { error } = await supabase.from('pbj-movies').update({ poster_path: `https://image.tmdb.org/t/p/original${data.poster_path}` }).eq('tmdb_id', movie.tmdb_id)
      } else if (!movie.release_date) {
        const { error } = await supabase.from('pbj-movies').update({ release_date: data.release_date.slice(0, 4) }).eq('tmdb_id', movie.tmdb_id)
      } else if (!movie.runtime) {
        const { error } = await supabase.from('pbj-movies').update({ runtime: data.runtime }).eq('tmdb_id', movie.tmdb_id)
      } else if (!movie.genres) {
        const { error } = await supabase.from('pbj-movies').update({ genres: data.genres }).eq('tmdb_id', movie.tmdb_id)
      }
    }
  }

  async function getDirector (movie) {
    if (!movie.director) {
      const movieDirector = movie.credits.crew.find(person => person.job
        === 'Director').name

      const { error } = await supabase.from('pbj-movies').update({ director: movieDirector }).eq('tmdb_id', movie.tmdb_id)
    }
  }
  async function getOscars (movie) {
    if (!movie.oscars) {
      const response = await fetch(`https://web-production-b8145.up.railway.app/awards/tmdb/${movie.tmdb_id}`).catch(error => console.error(error))
      if (!response) {
        throw new Error('Could not fetch Oscar noms and awards.')
      }
      const data = await response.json()
      // console.log(data)

      const { error } = await supabase.from('pbj-movies').update({ oscars: data }).eq('tmdb_id', movie.tmdb_id)
    }
  }
  function analyzeOscars (data) {
    const wins = data.filter(item => item.isWinner === '1')
    const allNominations = data // All entries are nominations (wins are nominations that won)

    // Get categories for all nominations
    const nominationCategories = allNominations.map(item => item.category)

    // Get acting nominations with names
    const actingNominations = allNominations
      .filter(item => item.isActing === '1')
      .map(item => ({
        category: item.category,
        names: item.names.map(n => n.name),
      }))

    // Get categories for wins
    const winCategories = wins.map(item => item.category)

    // Get acting wins with names
    const actingWins = wins
      .filter(item => item.isActing === '1')
      .map(item => ({
        category: item.category,
        names: item.names.map(n => n.name),
      }))

    return {
      totalNominations: allNominations.length,
      nominationCategories: nominationCategories,
      actingNominations: actingNominations,
      totalWins: wins.length,
      winCategories: winCategories,
      actingWins: actingWins,
    }
  }

  function formatOscarResults (results) {
    let output = '=== OSCAR ANALYSIS ===\n\n'

    output += `Total Nominations: ${results.totalNominations}\n`
    output += 'Categories:\n'
    for (const [i, cat] of results.nominationCategories.entries()) {
      output += `  ${i + 1}. ${cat}\n`
    }

    if (results.actingNominations.length > 0) {
      output += '\nActing Nominations:\n'
      for (const nom of results.actingNominations) {
        output += `  • ${nom.category}: ${nom.names.join(', ')}\n`
      }
    }

    output += `\nTotal Wins: ${results.totalWins}\n`
    if (results.totalWins > 0) {
      output += 'Categories:\n'
      for (const [i, cat] of results.winCategories.entries()) {
        output += `  ${i + 1}. ${cat}\n`
      }

      if (results.actingWins.length > 0) {
        output += '\nActing Wins:\n'
        for (const win of results.actingWins) {
          output += `  • ${win.category}: ${win.names.join(', ')}\n`
        }
      }
    } else {
      output += 'No wins\n'
    }

    return output
  }

  onMounted(() => {
    getMovies()
  })
</script>

<style scoped>
</style>
