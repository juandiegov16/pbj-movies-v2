<template>
  <div class="my-4 text-center justify-center ">
    <h1 class="text-h4 font-weight-bold">PBJ FILMS (v2)</h1>
  </div>  <!-- <v-img class="mb-4" height="150" src="@/assets/logo.png" /> -->
  <v-container fluid>
    <v-row>
      <v-col class="d-flex flex-wrap">
        <v-card
          v-for="movie in movies"
          :key="movie.tmdb_id"
          align="center"
          class="mx-auto"
        >
          <v-img v-if="movie.poster_path" height="200px" :src="movie.poster_path" />
          <v-card-title>{{ movie.title }}</v-card-title>
          <v-card-subtitle>{{
            movie.credits.crew.find(person => person.job ===
              "Director").name }}
          </v-card-subtitle>

          <div class="d-flex ga-12 justify-center mb-2">
            <v-badge
              bordered
              color="success"
              :content="movie.date_watched"
              inline
              location="right center"
            >
              <v-icon icon="mdi-calendar" />
            </v-badge>
          </div>

          <v-card-actions class="justify-center">
            <v-btn color="orange-lighten-2" text="Show Cast" />
            <v-spacer />
            <v-btn :icon="show ? 'mdi-chevron-up' : 'mdi-chevron-down'" @click="show = !show" />

          </v-card-actions>

          <v-expand-transition>
            <div v-show="show">
              <v-divider />

              <v-list-item
                v-for="person in movie.credits.cast.slice(0, 9)"
                :key="person.id"
                :subtitle="person.character"
                :title="person.name"
              />
            </div>
          </v-expand-transition>

          <v-list-item />
        </v-card>
      </v-col>
    </v-row></v-container>

</template>

<script setup>
  import { supabase } from '../utils/supabase'
  const movies = ref([])
  const show = ref(false)
  // const page = ref(1)
  // const itemsPerPage = ref(5)

  // const pageCount = computed(() => {
  //   return Math.ceil(movies.value.length / itemsPerPage.value)
  // })

  async function getMovies () {
    const { data } = await supabase.from('pbj-movies').select().order('date_watched', { ascending: false }).range(0, 5)
    movies.value = data

    for (const movie of movies.value) {
      getPoster(movie)
    }
  }

  async function getPoster (movie) {
    if (!movie.poster_path) {
      const options = {
        method: 'GET',
        headers: {
          accept: 'application/json',
          Authorization: 'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJiNTY5Y2Y4ZTIyZTlmYTRiOWZkMGQzY2RhNzdlOGExZSIsIm5iZiI6MTc2MDcyMjE3Mi4wMjMsInN1YiI6IjY4ZjI3Y2ZjZjg4NzEyMjg3ZmVjNWMxZCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.GljyKUapeItmVRL7SgnNHWU0Z9WacUn2MN0rfQFcD7w',
        },
      }

      const response = await fetch(`https://api.themoviedb.org/3/movie/${movie.tmdb_id}?language=en-US`, options)
        // .then(res => res.json())
        // .then(res => console.log(res))
        .catch(error => console.error(error))

      if (!response.ok) {
        throw new Error('Could not fetch resource.')
      }

      const data = await response.json()
      console.log(data)
      console.log(data.poster_path)
      console.log(`https://image.tmdb.org/t/p/original${data.poster_path}`)


      const { error } = await supabase.from('pbj-movies').update({ poster_path: `https://image.tmdb.org/t/p/original${data.poster_path}` }).eq('tmdb_id', movie.tmdb_id)
    }
  }

  onMounted(() => {
    getMovies()
  })
</script>
