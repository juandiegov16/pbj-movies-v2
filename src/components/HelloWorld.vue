<template>
  <v-container class="fill-height" max-width="900">
    <div>
      <v-img class="mb-4" height="150" src="@/assets/logo.png" />

      <div class="mb-8 text-center">
        <div class="text-body-2 font-weight-light mb-n1">Welcome to</div>
        <h1 class="text-h2 font-weight-bold">PBJ FILMS (v2)</h1>
      </div>

      <!-- <v-row>
        <v-col v-for="movie in movies" :key="movie.tmdb_id" cols="3">
          <v-card class="mx-auto" rounded="lg" :title="movie.title">
            <v-divider />
            <v-card-text>
              <span><i><b>Director:</b> {{movie.credits.crew.find(person => person.job === "Director").name}}</i>
                <br></span>
              <v-badge bordered color="success" :content="movie.date_watched" inline location="top right">
                <v-icon icon="mdi-calendar" />
              </v-badge>
              <v-divider />
              <ul>
                <li v-for="person in movie.credits.cast.slice(0, 9)" :key="person.id">
                  {{ person.name }} <span class="text-gray-500">as</span> {{ person.character }}
                </li>
              </ul>
            </v-card-text>

          </v-card>
        </v-col>
      </v-row> -->

      <v-timeline v-for="movie in movies" :key="movie.tmdb_id">
        <v-timeline-item align="center" class="mb-4">
          <template #opposite>
            <div class="text-h5" v-text="movie.title" />
            <v-badge
              bordered
              color="success"
              :content="movie.date_watched"
              inline
              location="top right"
            >
              <v-icon icon="mdi-calendar" />
            </v-badge><div>
              <p class="text-button">Director:</p> <i> {{ movie.credits.crew.find(person => person.job === "Director").name }}</i>
            </div>
          </template>
          <!-- <ul class="text-body-1">
            <li v-for="person in movie.credits.cast.slice(0, 9)" :key="person.id" class="text-body-1 text-left">
              {{ person.name }} <span class="text-gray-500">as</span> {{ person.character }}
            </li>
          </ul> -->

          <v-card class="mx-auto" max-width="300">
            <v-list-item
              v-for="person in movie.credits.cast.slice(0, 9)"
              :key="person.id"
              :subtitle="person.character"
              :title="person.name"
            />
          </v-card>
        </v-timeline-item>

      </v-timeline>

    </div>
  </v-container>
</template>

<script setup>
  import { supabase } from '../utils/supabase'
  const movies = ref([])

  async function getMovies () {
    const { data } = await supabase.from('pbj-movies').select().order('date_watched', { ascending: false })
    movies.value = data
  }

  onMounted(() => {
    getMovies()
  })
</script>
