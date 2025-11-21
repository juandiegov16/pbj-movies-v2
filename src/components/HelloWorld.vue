<template>
  <v-container class="fill-height max-width">
    <div class="mb-8 text-center justify-center d-flex">
      <h1 class="text-h4 font-weight-bold">PBJ FILMS (v2)</h1>
    </div>
    <div class="d-flex justify-center" :items-per-page="itemsPerPage" :page="page">
      <!-- <v-img class="mb-4" height="150" src="@/assets/logo.png" /> -->

      <v-timeline v-for="movie in movies" :key="movie.tmdb_id" align="center" direction="horizontal">
        <v-timeline-item class="md-4">
          <template #opposite>
            <v-card class="mx-auto" :title="movie.title" align="center">
              <div class="d-flex ga-12 justify-center mb-2"><v-badge
                bordered
                color="success"
                :content="movie.date_watched"
                inline
                location="right center"
              >
                <v-icon icon="mdi-calendar" />
              </v-badge></div><v-card-subtitle>{{ movie.credits.crew.find(person => person.job ===
                "Director").name }}</v-card-subtitle>

            </v-card>

          </template>

          <v-card class="mx-auto">
            <v-list-item
              v-for="person in movie.credits.cast.slice(0, 9)"
              :key="person.id"
              :subtitle="person.character"
              :title="person.name"
            />
          </v-card>
        </v-timeline-item>
        <template #bottom>
          <div class="text-center pt-2">
            <v-pagination v-model="page" :length="pageCount" :total-visible="7" />
          </div>
        </template>
      </v-timeline>

    </div>
  </v-container>
</template>

<script setup>
  import { supabase } from '../utils/supabase'
  const movies = ref([])
  const page = ref(1)
  const itemsPerPage = ref(5)

  const pageCount = computed(() => {
    return Math.ceil(movies.value.length / itemsPerPage.value)
  })

  async function getMovies () {
    const { data } = await supabase.from('pbj-movies').select().order('date_watched', { ascending: false }).range(0, 4)
    movies.value = data
  }

  onMounted(() => {
    getMovies()
  })
</script>
