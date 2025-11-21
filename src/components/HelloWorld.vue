<template>
  <v-container class="fill-height" max-width="1200">
    <div>
      <v-img class="mb-4" height="150" src="@/assets/logo.png" />

      <div class="mb-8 text-center">
        <div class="text-body-2 font-weight-light mb-n1">Welcome to</div>
        <h1 class="text-h2 font-weight-bold">PBJ FILMS (v2)</h1>
      </div>

      <v-row>
        <v-col v-for="movie in movies" :key="movie.tmdb_id" cols="3">
          <v-card class="mx-auto" rounded="lg" :title="movie.title">
            <v-divider />
            <v-card-text>
              <span><i><b>Director:</b> {{movie.credits.crew.find(person => person.job === "Director").name}}</i>
                <br></span>
              <span><i><b>Date watched:</b> {{ movie.date_watched }}</i> </span>
              <v-divider />
              <ul>
                <li v-for="person in movie.credits.cast.slice(0, 9)" :key="person.id">
                  {{ person.name }} <span class="text-gray-500">as</span> {{ person.character }}
                </li>
              </ul>
              <!-- <v-divider />

               -->
            </v-card-text>

          </v-card>
        </v-col>
      </v-row>

    </div>
  </v-container>
</template>

<script setup>
import { supabase } from '../utils/supabase'
const movies = ref([])

async function getMovies() {
  const { data } = await supabase.from('pbj-movies').select().order('date_watched', { ascending: false })
  movies.value = data
}

onMounted(() => {
  getMovies()
})
</script>
