<template>
  <v-container class="fill-height" max-width="900">
    <div>
      <v-img
        class="mb-4"
        height="150"
        src="@/assets/logo.png"
      />

      <div class="mb-8 text-center">
        <div class="text-body-2 font-weight-light mb-n1">Welcome to</div>
        <h1 class="text-h2 font-weight-bold">PBJ FILMS (v2)</h1>
      </div>

      <v-row>
        <v-col>
          <ul>
            <li v-for="movie in movies" :key="movie.tmdb_id">{{ movie.title }}</li>
          </ul>
        </v-col>
      </v-row>

    </div>
  </v-container>
</template>

<script setup>
  import { supabase } from '../utils/supabase'
  const movies = ref([])

  async function getMovies () {
    const { data } = await supabase.from('pbj-movies').select()
    movies.value = data
  }

  onMounted(() => {
    getMovies()
  })
</script>
