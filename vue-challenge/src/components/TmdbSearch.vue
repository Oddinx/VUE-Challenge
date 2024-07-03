<template>
  <div>
    <nav class="navbar">
      <h1>TMDb Movie Search</h1>
      <div class="search-container">
        <div class="search-wrapper">
          <input
            v-model="searchQuery"
            @keyup.enter="searchMovies"
            placeholder="Search for a movie..."
          />
          <button v-if="searchQuery" @click="clearSearch" class="clear-button">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24">
              <path
                fill="currentColor"
                d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12 19 6.41z"
              ></path>
            </svg>
          </button>
        </div>
        <button @click="searchMovies" class="search-button">
          <svg class="search-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
            <path
              fill="currentColor"
              d="M10 2C5.58 2 2 5.58 2 10s3.58 8 8 8c1.66 0 3.21-.54 4.47-1.46l5.57 5.57 1.42-1.42-5.57-5.57C17.46 13.21 18 11.66 18 10c0-4.42-3.58-8-8-8zm0 2c3.31 0 6 2.69 6 6s-2.69 6-6 6-6-2.69-6-6 2.69-6 6-6z"
            ></path>
          </svg>
        </button>
        <button @click="showModal = true" class="filter-button">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" width="24" height="24">
            <path
              fill="currentColor"
              d="M3.9 54.9C10.5 40.9 24.5 32 40 32H472c15.5 0 29.5 8.9 36.1 22.9s4.6 30.5-5.2 42.5L320 320.9V448c0 12.1-6.8 23.2-17.7 28.6s-23.8 4.3-33.5-3l-64-48c-8.1-6-12.8-15.5-12.8-25.6V320.9L9 97.3C-.7 85.4-2.8 68.8 3.9 54.9z"
            />
          </svg>
        </button>
      </div>
    </nav>

    <div v-if="loading">Loading...</div>
    <div class="movies-container" v-if="movies.length">
      <div class="movie-card" v-for="movie in movies" :key="movie.id">
        <img :src="getImageUrl(movie.poster_path)" alt="Movie Poster" v-if="movie.poster_path" />
        <h2>{{ movie.title }}</h2>
        <p>Release Date: {{ movie.release_date }}</p>
        <p>{{ movie.overview }}</p>
      </div>
    </div>

    <!-- Modal -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="showModal = false">&times;</span>
        <h2>Select Sort Order</h2>
        <button @click="applyFilter('release_date.desc')">Release Date (Newest First)</button>
        <button @click="applyFilter('release_date.asc')">Release Date (Oldest First)</button>
        <button @click="applyFilter('original_title.asc')">Title (A-Z)</button>
        <button @click="applyFilter('original_title.desc')">Title (Z-A)</button>
      </div>
    </div>
  </div>
</template>
<script>
import axios from 'axios'

export default {
  data() {
    return {
      apiKey: 'c91aa70fac6a724f535c51022662c855',
      searchQuery: '',
      sortBy: 'release_date.desc',
      movies: [],
      loading: false,
      showModal: false
    }
  },
  methods: {
    async searchMovies() {
      if (this.searchQuery.trim() === '') return

      this.loading = true
      const url = `https://api.themoviedb.org/3/search/movie?api_key=${this.apiKey}&query=${encodeURIComponent(this.searchQuery)}`

      try {
        const response = await axios.get(url)
        this.movies = response.data.results
        this.sortMovies()
      } catch (error) {
        console.error('Error fetching movies:', error)
      } finally {
        this.loading = false
      }
    },
    sortMovies() {
      if (this.sortBy === 'release_date.desc') {
        this.movies.sort((a, b) => (a.release_date < b.release_date ? 1 : -1))
      } else if (this.sortBy === 'release_date.asc') {
        this.movies.sort((a, b) => (a.release_date > b.release_date ? 1 : -1))
      } else if (this.sortBy === 'original_title.asc') {
        this.movies.sort((a, b) => (a.title > b.title ? 1 : -1))
      } else if (this.sortBy === 'original_title.desc') {
        this.movies.sort((a, b) => (a.title < b.title ? 1 : -1))
      }
    },
    getImageUrl(path) {
      return `https://image.tmdb.org/t/p/w500${path}`
    },
    applyFilter(sortOption) {
      this.sortBy = sortOption
      this.sortMovies()
      this.showModal = false
    },
    clearSearch() {
      this.searchQuery = ''
    }
  }
}
</script>
