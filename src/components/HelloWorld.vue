<template>
  <div class="my-4 text-center justify-center ">
    <h1 class="text-h4 font-weight-bold">PBJ FILMS (v2)</h1>
  </div>  <!-- <v-img class="mb-4" height="150" src="@/assets/logo.png" /> -->
  <v-container fluid>
    <v-row no-gutters>
      <v-col class="d-flex flex-wrap">
        <v-card
          v-for="movie in movies"
          :key="movie.tmdb_id"
          align="center"
          class="mx-auto"
        >

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
    const { data } = await supabase.from('pbj-movies').select().order('date_watched', { ascending: false }).range(0, 4)
    movies.value = data
  }

  onMounted(() => {
    getMovies()
  })
</script>
