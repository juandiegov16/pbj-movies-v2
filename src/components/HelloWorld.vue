<template>
  <div class="my-4 text-center justify-center ">
    <h1 class="text-h4 font-weight-bold">PBJ FILMS (v2) </h1>
    <!-- <FlagIcon circle code="us" size="32" /> -->
  </div>

  <v-container fluid>
    <v-row align="center" class="fill-height" justify="center">

      <v-col class="d-flex flex-wrap" cols="12" justify="around">

        <v-card v-for="(movie, index) in paginatedMovies" :key="movie.tmdb_id" align="center" class="mx-auto">
          <v-img
            v-if="movie.poster_path"
            class="mx-auto pa-2 ma-2"
            cover
            height="450px"
            :src="movie.poster_path"
            width="300px"
          />

          <v-card-subtitle v-if="movie.director"><v-badge :content="movie.director" inline><v-icon
            icon="mdi-movie-open"
          /></v-badge>
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

            <v-btn v-if="movie.oscars?.nominations?.length" class="text-teal" icon="mdi-trophy" @click="onShowOscarsClick(movie)" />

            <v-dialog v-model="oscarsDialog" max-width="500">
              <v-card v-if="selectedMovieForOscars">
                <v-card-title>{{ selectedMovieForOscars.title }} ({{ selectedMovieForOscars.release_date }}): Oscars</v-card-title>

                <v-card-text>
                  <v-list lines="two">
                    <v-list-item v-for="g in groupedNominations" :key="g.key">
                      <template #prepend>
                        <v-icon
                          :color="g.winner ? 'amber' : undefined"
                          :icon="g.winner ? 'mdi-trophy' : 'mdi-trophy-outline'"
                        />
                      </template>

                      <v-list-item-title>{{ g.category }}</v-list-item-title>

                      <v-list-item-subtitle>
                        {{ g.ceremony_year }} ceremony · {{ g.winner ? 'Won' : 'Nominated' }}
                        <br>
                        {{ g.nominees.join(', ') }}
                      </v-list-item-subtitle>
                    </v-list-item>
                  </v-list>
                </v-card-text>

                <v-card-actions>
                  <v-spacer />
                  <v-btn text="Close" @click="oscarsDialog = false" />
                </v-card-actions>
              </v-card>
            </v-dialog>

          </v-card-actions>

          <div class="d-flex justify-center ga-2 mt-2">
            <v-chip v-for="genre in movie.genres" :key="genre.id" color="secondary" variant="flat">{{ genre.name
            }}</v-chip>
          </div>

          <v-dialog v-model="castDialog" max-width="500">
            <v-card v-if="selectedMovieForCast" prepend-icon="mdi-account-group" :subtitle="selectedMovieForCast.director" :title="selectedMovieForCast.title">
              <v-list-item
                v-for="person in selectedMovieForCast.credits.cast.slice(0, 7)"
                :key="person.id"
                :subtitle="person.character"
                :title="person.name"
              />

              <v-card-actions>
                <v-spacer />
                <v-btn text="Close" @click="castDialog = false" />
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-list-item />
        </v-card>
      </v-col>

    </v-row>

    <v-pagination v-model="page" class="my-4" :length="pageLength" :total-visible="itemsPerPage" />

    <button :disabled="loadingOscars || movies.length === 0" @click="loadOscars">
      Load Oscars
    </button>

  </v-container>

</template>

<script setup>
  // import FlagIcon from 'vue3-flag-icons'
  import { supabase } from '../utils/supabase'
  const movies = ref([])
  const page = ref(1)
  const itemsPerPage = 10
  const pageLength = computed(() => {
    return Math.ceil(movies.value.length / itemsPerPage)
  })
  const loadingOscars = ref(false)
  const oscarsDialog = ref(false)
  const castDialog = ref(false)
  const selectedMovieForCast = ref(null)
  const selectedMovieForOscars = ref(null)

  async function loadOscars () {
    loadingOscars.value = true
    try {
      const failed = await getOscars(movies.value)
      if (failed.length > 0) console.warn('Some movies failed:', failed)
    } finally {
      loadingOscars.value = false
    }
  }

  async function getMovies () {
    const { data } = await supabase.from('pbj-movies').select().order('date_watched', { ascending: false })
    movies.value = data

    for (const movie of movies.value) {
      getCredits(movie)
      getDetails(movie)
      getDirector(movie)
    }

  // console.log(pageLength.value)
  }

  function onShowCastClick (movie) {
    selectedMovieForCast.value = movie
    castDialog.value = true
  }

  function onShowOscarsClick (movie) {
    selectedMovieForOscars.value = movie
    oscarsDialog.value = true
  }
  const groupedNominations = computed(() =>
    selectedMovieForOscars.value ? groupNominations(selectedMovieForOscars.value.oscars) : [],
  )

  function groupNominations (oscars) {
    const noms = oscars?.nominations ?? []
    const groups = {}

    for (const n of noms) {
      const key = `${n.ceremony_year}|${n.category}`
      if (!groups[key]) {
        groups[key] = {
          key,
          category: n.category,
          ceremony_year: n.ceremony_year,
          winner: false,
          nominees: [],
        }
      }
      groups[key].winner ||= n.winner
      groups[key].nominees.push(n.nominee)
    }

    return Object.values(groups).toSorted(
      (a, b) => b.winner - a.winner || a.ceremony_year - b.ceremony_year || a.category.localeCompare(b.category),
    )
  }

  function movieNumber (index) {
    return movies.value.length - (index + ((page.value - 1) * itemsPerPage))
  }

  async function getCredits (movie) {
    if (!movie.credits) {
      const options = {
        method: 'GET',
        headers: {
          accept: 'application/json',
          Authorization: 'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJiNTY5Y2Y4ZTIyZTlmYTRiOWZkMGQzY2RhNzdlOGExZSIsIm5iZiI6MTc2MDcyMjE3Mi4wMjMsInN1YiI6IjY4ZjI3Y2ZjZjg4NzEyMjg3ZmVjNWMxZCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.GljyKUapeItmVRL7SgnNHWU0Z9WacUn2MN0rfQFcD7w',
        },
      }

      const response = await fetch(`https://api.themoviedb.org/3/movie/${movie.tmdb_id}/credits`, options)
        .catch(error => console.error(error))

      if (!response) {
        throw new Error('Could not fetch resource.')
      }

      const data = await response.json()
      if (!movie.credits) {
        const { error } = await supabase.from('pbj-movies').update({ credits: data }).eq('tmdb_id', movie.tmdb_id)
      }
    }
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
  const sleep = ms => new Promise(r => setTimeout(r, ms))

  async function fetchJson (url, retries = 3) {
    for (let attempt = 0; attempt <= retries; attempt++) {
      const res = await fetch(url)

      if (res.status === 429) {
        const wait = Number(res.headers.get('Retry-After')) * 1000 || 1000 * 2 ** attempt
        await sleep(wait)
        continue
      }
      if (!res.ok) throw new Error(`Request failed (${res.status}): ${url}`)
      return res.json()
    }
    throw new Error(`Rate limited too many times: ${url}`)
  }

  async function getOscars (movies, delayMs = 300) {
    console.log(`getOscars: starting with ${movies.length} movies`)
    const failed = []
    let done = 0, skipped = 0

    for (const movie of movies) {
      if (movie.oscars) {
        skipped++
        console.log(`skipping TMDB ${movie.tmdb_id}, oscars already set:`, movie.oscars)
        continue
      }

      try {
        // Step 1: TMDB id -> OscarBase id
        const list = await fetchJson(`/api/movies?tmdb_id=${movie.tmdb_id}`)
        console.log(`search result for TMDB ${movie.tmdb_id}:`, list)
        const match = list.data?.[0] // adjust to the real response shape

        let oscars
        if (match) {
          await sleep(delayMs)
          const res = await fetchJson(`/api/movies/${match.id}`)
          oscars = res.data // store the inner object, not the { data } wrapper
        } else {
          oscars = { nominations: [], no_oscar_data: true }
        }

        // Save to Supabase
        const { data, error } = await supabase
          .from('pbj-movies')
          .update({ oscars })
          .eq('tmdb_id', movie.tmdb_id)
          .select('tmdb_id') // returns the rows that were actually updated

        if (error) throw error
        if (!data?.length) {
          throw new Error(`Update matched 0 rows for tmdb_id ${movie.tmdb_id} (check RLS policy and id type)`)
        }

        movie.oscars = oscars // keep local state in sync
        done++
        console.log(`saved TMDB ${movie.tmdb_id} (${done} done)`)
      } catch (error) {
        // Real failures (network, rate limit, DB error) are NOT marked,
        // so they get retried on the next run
        console.error(`getOscars failed for TMDB ${movie.tmdb_id}:`, error.message)
        failed.push({ tmdb_id: movie.tmdb_id, reason: error.message })
      }

      await sleep(delayMs) // pause before the next movie
    }

    console.log(`getOscars finished: ${done} saved, ${skipped} skipped, ${failed.length} failed`)
    return failed
  }

  const paginatedMovies = computed(() => {
    const start = (page.value - 1) * itemsPerPage
    const end = start + itemsPerPage
    return movies.value.slice(start, end)
  })

  onMounted(() => {
    getMovies()
    loadOscars()
  })
</script>

<style scoped></style>
