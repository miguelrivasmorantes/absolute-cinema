<script lang="ts">
  import Card from '$lib/components/Card.svelte';

  type Movie = {
    id: number;
    titulo: string;
    sinopsis: string;
    posters: string;
  };

  let movies: Movie[] = [];

  async function loadMovies() {
    const response = await fetch('http://127.0.0.1:8000/api/peliculas');
    const data = await response.json();
    movies = data.peliculas;
  }

  loadMovies();
</script>

<section class="space-y-4 mx-20">
  <h1 class="text-3xl font-bold">Movies</h1>

  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {#each movies as movie}
      <Card title={movie.titulo} description={movie.sinopsis} image={movie.posters} />
    {/each}
  </div>
</section>
