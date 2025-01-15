<template>
  <div id="app" class="kiosk">
    <div
      class="video-container"
      v-for="(movie, index) in movies"
      :key="index"
      :class="{ fadeIn: index === currentMovieIndex, fadeOut: index !== currentMovieIndex }"
    >
      <iframe
        :src="getTrailerUrl(movie.trailerKey)"
        width="560"
        height="315"
        v-show="index === currentMovieIndex"
        ref="currentIframe"
        frameborder="0"
        allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
        allowfullscreen
        class="background-video"
      ></iframe>
      <h1 class="movie-title">{{ movie.title }}</h1>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      movies: [],
      currentMovieIndex: 0,
    };
  },
  computed: {
    currentMovie() {
      return this.movies[this.currentMovieIndex] || null;
    },
  },
  methods: {
    async fetchMovies() {
      const options = {
        method: 'GET',
        headers: {
          accept: 'application/json',
          Authorization: 'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJkMzFkYjM0NWQ1NzUwNDk0ZjE1MDgzNWQ4MTA5MzIyNiIsIm5iZiI6MTczNjUwNTA0OS42MzEsInN1YiI6IjY3ODBmNmQ5MTQzMWUwNTkxYWJiMzEzYSIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.vQl2-EAIqO5GpQwdp0cxzrXHbQU_9pCgaaq_aFEZUiw'
        }
      };


      try {
        const response = await fetch(
          "https://api.themoviedb.org/3/movie/now_playing?language=en-US",
          options
        );
        const data = await response.json();
        const movieList = data.results.slice(0,5) || [];

        await Promise.all(
          movieList.map(async (movie) => {
            const videoResponse = await fetch(
              `https://api.themoviedb.org/3/movie/${movie.id}/videos?language=en-US`,
              options
            );
            const videoData = await videoResponse.json();
            const trailer = videoData.results.find(
              (video) => video.type === "Trailer" && video.site === "YouTube"
            );
            movie.trailerKey = trailer ? trailer.key : "";
          })
        );

        this.movies = movieList;
        this.startCarousel();
      } catch (error) {
        console.error("Error fetching movies:", error);
      }
    },
    getTrailerUrl(trailerKey) {
      return `https://www.youtube.com/embed/${trailerKey}?controls=0&autoplay=1&mute=1&loop=1&playlist=${trailerKey}`;
    },
    startCarousel() {
      setInterval(() => {
        // Pause the current iframe before switching
        const currentIframe = this.$refs.currentIframe[this.currentMovieIndex];
        if (currentIframe && currentIframe.contentWindow) {
          currentIframe.contentWindow.postMessage(
            '{"event":"command","func":"pauseVideo","args":""}',
            "*"
          );
        }

        // Update movie index
        this.currentMovieIndex = (this.currentMovieIndex + 1) % this.movies.length;
      }, 15000); // Change movie every 15 seconds
    },
  },
  mounted() {
    this.fetchMovies();
  },
};
</script>

<style>
.kiosk {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: black;
  color: white;
  position: relative;
}

.video-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  transition: opacity 1.5s ease-in-out;
}

.video-container.fadeIn {
  opacity: 1;
}

.background-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.movie-title {
  position: absolute;
  bottom: 20px;
  left: 20px;
  font-size: 3rem;
  font-weight: bold;
}
</style>
