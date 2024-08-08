<template>
  <h1>Kpoint Movie App</h1>
  <div v-for="item in genresList" class="btn-row">
    <button
      :class="[
        selectedGenras.includes(item.id) ? 'filter-btn-active' : 'filter-btn',
      ]"
      @click="selectedBtn(item.id)"
    >
      {{ item.name }}
    </button>
  </div>
  <div v-for="year in years">
    <h5 style="text-align: start; padding-left: 100px">{{ year }}</h5>
    <div class="card">
      <div v-for="movie in movieList" class="card-details">
        <div class="caption">
          <h2>{{ movie.title }}</h2>
          <p>{{ movie.overview }}</p>
        </div>
        <img
          :src="'https://image.tmdb.org/t/p/w500' + movie.poster_path"
          alt="movie.title"
        />
      </div>
      <div v-if="loading">Loading...</div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      movieList: [],
      page: 1,
      loading: false,
      genre: "",
      selectedGenras: [],
      genresList: [],
      years: [],
    };
  },
  mounted() {
    this.fetchGeneras();
    this.fetchMovies();
    window.addEventListener("scroll", this.handleScroll);
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.handleScroll);
  },
  methods: {
    async fetchGeneras() {
      const res = await axios.get(
        "https://api.themoviedb.org/3/genre/movie/list",
        {
          params: {
            api_key: "29bec32e5d45e6f9bcb0d30faaf022f5",
          },
        }
      );
      this.genresList = res.data.genres;
      console.log(res);
    },
    selectedBtn(value) {
      this.genre = value;
      if (this.selectedGenras.includes(value)) {
        this.selectedGenras = this.selectedGenras.filter(
          (item) => item !== value
        );
      } else {
        this.selectedGenras.push(value);
      }
      this.movieList = [];
      this.page = 1;
      this.loading = true;
      this.fetchMovies();
    },
    async fetchMovies() {
      try {
        const res = await axios.get(
          "https://api.themoviedb.org/3/discover/movie",
          {
            params: {
              api_key: "29bec32e5d45e6f9bcb0d30faaf022f5",
              sort_by: "popularity.desc",
              page: this.page,
              primary_release_year: this.years,
              with_genres: this.selectedGenras,
              vote_count_gte: 100,
            },
          }
        );
        this.movieList = [...this.movieList, ...res.data.results];
        const releaseDate = this.movieList.map((item) => {
          const year = item.release_date.split("-");
          return year[0];
        });
        this.years = [...new Set(releaseDate)];
        console.log(this.years);
      } catch (error) {
        console.log(error);
      } finally {
        this.loading = false;
      }
    },
    handleScroll() {
      if (
        window.innerHeight + window.scrollY >=
          document.body.offsetHeight - 500 &&
        !this.loading
      ) {
        this.loading = true;
        this.page += 1;
        this.fetchMovies();
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.filter-btn {
  background: #000;
  color: #fff;
  padding: 5px 10px;
  border-radius: 5px;
  border: 1px solid #000;
}
.filter-btn-active {
  background: red;
  color: #fff;
  border: 1px solid red;
  padding: 5px 10px;
  border-radius: 5px;
}
.btn-row {
  display: inline-flex;
  flex-direction: column;
  justify-content: center;
  flex-wrap: wrap;
  margin: 2px;
}
.card {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
}
.card img {
  height: 250px;
}
.card p {
  font-size: 12px;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-details {
  background-color: #ffff;
  margin: 10px;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  text-align: center;
}
.caption {
  background: #fff;
  color: #000;
  opacity: 0.5;
  bottom: 0;
  position: absolute;
}
</style>
