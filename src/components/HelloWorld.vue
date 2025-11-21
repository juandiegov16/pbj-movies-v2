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
        <v-col cols="12">
          <v-card
            class="py-4"
            color="surface-variant"
            image="https://cdn.vuetifyjs.com/docs/images/one/create/feature.png"
            prepend-icon="mdi-rocket-launch-outline"
            rounded="lg"
            variant="tonal"
          >
            <template #image>
              <v-img position="top right" />
            </template>

            <template #title>
              <h2 class="text-h5 font-weight-bold">
                Get started
              </h2>
            </template>

            <template #subtitle>
              <div class="text-subtitle-1">
                Change this page by updating <v-kbd>{{ `<HelloWorld />` }}</v-kbd> in <v-kbd>components/HelloWorld.vue</v-kbd>.
              </div>
            </template>
          </v-card>
        </v-col>
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
