<template>
  <div class="my-4 text-center justify-center ">
    <h1 class="text-h4 font-weight-bold">PBJ FILMS (v2) </h1>
    <!-- <FlagIcon circle code="us" size="32" /> -->
  </div>

  <div v-if="selectedGenres.length > 0" class="d-flex align-center ga-2 mb-4">
    <span class="text-caption">Filtering by:</span>

    <v-chip
      v-for="id in selectedGenres"
      :key="id"
      closable
      color="secondary"
      @click:close="toggleGenre(id)"
    >
      {{ genreNames[id] }}
    </v-chip>

    <v-btn size="small" text="Clear all" variant="text" @click="selectedGenres = []" />
  </div>

  <v-container fluid>
    <div class="movie-grid">
      <v-card v-for="(movie, index) in paginatedMovies" :key="movie.tmdb_id" class="movie-card">
        <v-img
          v-if="movie.poster_path"
          class="mx-auto pa-2 ma-2"
          cover
          height="450px"
          :src="movie.poster_path"
          width="300px"
        />

        <div class="d-flex justify-center">
          <v-card-subtitle v-if="movie.director">
            <v-badge :content="movie.director" inline>
              <v-icon icon="mdi-movie-open" />
            </v-badge>
          </v-card-subtitle>
        </div>

        <div class="d-flex flex-wrap justify-center align-center ga-2 mt-1">
          <v-badge v-if="movie.release_date" color="primary" :content="movie.release_date" inline>
            <v-icon icon="mdi-cake" />
          </v-badge>

          <v-badge color="warning" :content="`${movie.runtime} min`" inline>
            <v-icon icon="mdi-clock" />
          </v-badge>

          <v-badge
            bordered
            color="success"
            :content="movie.date_watched"
            inline
            location="right center"
          >
            <v-icon icon="mdi-calendar" />
          </v-badge>

          <v-badge color="secondary" :content="movieNumber(index)" inline>
            <v-icon icon="mdi-counter" />
          </v-badge>
        </div>

        <v-card-actions class="justify-center">
          <v-btn class="text-amber" icon="mdi-account-group" @click="onShowCastClick(movie)" />

          <v-btn
            v-if="movie.oscars?.nominations?.length"
            class="text-teal"
            icon="mdi-trophy"
            @click="onShowOscarsClick(movie)"
          />
        </v-card-actions>

        <div class="d-flex flex-wrap justify-center ga-2 mt-auto pt-2 pb-3">
          <v-chip
            v-for="genre in movie.genres"
            :key="genre.id"
            :aria-pressed="isGenreActive(genre.id)"
            class="genre-badge"
            :class="{ 'genre-badge--off': selectedGenres.length > 0 && !isGenreActive(genre.id) }"
            color="secondary"
            role="button"
            tabindex="0"
            :text="genre.name"
            @click.stop="toggleGenre(genre.id)"
            @keydown.enter.stop="toggleGenre(genre.id)"
          />
        </div>
      </v-card>
    </div>

    <!-- dialogs: once, outside the grid and the v-for -->
    <v-dialog v-model="oscarsDialog" max-width="500">
      <v-card v-if="selectedMovieForOscars">
        <v-card-title>
          {{ selectedMovieForOscars.title }} ({{ selectedMovieForOscars.release_date }}): Oscars
        </v-card-title>

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

    <v-dialog v-model="castDialog" max-width="500">
      <v-card
        v-if="selectedMovieForCast"
        prepend-icon="mdi-account-group"
        :subtitle="selectedMovieForCast.director"
        :title="selectedMovieForCast.title"
      >
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

    <v-pagination v-model="page" class="my-4" :length="pageLength" :total-visible="itemsPerPage" />

    <button :disabled="loadingOscars || movies.length === 0" @click="loadOscars">
      Load Oscars
    </button>

  </v-container>

</template>

<script setup>
  import { watch } from 'vue'
  // import FlagIcon from 'vue3-flag-icons'
  import { supabase } from '../utils/supabase'
  const movies = ref([])
  const page = ref(1)
  const itemsPerPage = 20
  const filteredMovies = computed(() => {
    if (selectedGenres.value.length === 0) return movies.value
    return movies.value.filter(m =>
      selectedGenres.value.every(id => m.genres?.some(g => g.id === id)),
    )
  })

  const pageLength = computed(() => {
    return Math.max(1, Math.ceil(filteredMovies.value.length / itemsPerPage))
  })
  const paginatedMovies = computed(() => {
    const start = (page.value - 1) * itemsPerPage
    const end = start + itemsPerPage
    return filteredMovies.value.slice(start, end)
  })
  const loadingOscars = ref(false)
  const oscarsDialog = ref(false)
  const castDialog = ref(false)
  const selectedMovieForCast = ref(null)
  const selectedMovieForOscars = ref(null)
  const selectedGenres = ref([]) // genre ids

  watch(selectedGenres, () => {
    page.value = 1
  })

  function isGenreActive (id) {
    return selectedGenres.value.includes(id)
  }

  function toggleGenre (id) {
    selectedGenres.value = isGenreActive(id)
      ? selectedGenres.value.filter(g => g !== id)
      : [...selectedGenres.value, id]
  }

  const genreNames = computed(() => {
    const map = {}
    for (const m of movies.value) for (const g of m.genres ?? []) map[g.id] = g.name
    return map
  })

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
    return filteredMovies.value.length - (index + ((page.value - 1) * itemsPerPage))
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
      const updates = {}

      if (!movie.poster_path && data.poster_path) {
        updates.poster_path = `https://image.tmdb.org/t/p/original${data.poster_path}`
      }
      if (!movie.release_date && data.release_date) {
        updates.release_date = data.release_date.slice(0, 4)
      }
      if (!movie.runtime && data.runtime) {
        updates.runtime = data.runtime
      }
      if (!movie.genres && data.genres) {
        updates.genres = data.genres
      }

      const fields = Object.keys(updates)

      if (fields.length === 0) {
        console.log(`TMDB ${movie.tmdb_id}: nothing to update`)
      } else {
        console.log(`TMDB ${movie.tmdb_id}: fetched ${fields.join(', ')}`)

        const { data: saved, error } = await supabase
          .from('pbj-movies')
          .update(updates)
          .eq('tmdb_id', movie.tmdb_id)
          .select('tmdb_id') // returns the rows actually updated

        if (error) {
          console.error(`TMDB ${movie.tmdb_id}: save failed`, error.message)
        } else if (saved?.length) {
          Object.assign(movie, updates) // keep local state in sync
          console.log(`TMDB ${movie.tmdb_id}: saved ${fields.join(', ')}`)
        } else {
          console.error(`TMDB ${movie.tmdb_id}: update matched 0 rows (check RLS policy and id type)`)
        }
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

  onMounted(() => {
    getMovies()
    loadOscars()
  })
</script>

<style scoped>
.movie-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 16px;
  padding: 16px;
}

.movie-card {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.genre-badge {
  cursor: pointer;
  user-select: none;
  transition: opacity 0.15s;
}
.genre-badge:hover {
  opacity: 0.85;
}
.genre-badge--off {
  opacity: 0.4;
}

</style>
