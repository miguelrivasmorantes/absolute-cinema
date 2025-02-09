<script>
  import Home from './routes/Home.svelte';
  import Movies from './routes/Movies.svelte';
  import Actors from './routes/Actors.svelte';

  const views = {
    home: Home,
    movies: Movies,
    actors: Actors,
  };

  let currentView = 'Home';
</script>

<nav>
  <button on:click={() => currentView = 'home'}>Home</button>
  <button on:click={() => currentView = 'movies'}>Movies</button>
  <button on:click={() => currentView = 'actors'}>Actors</button>
</nav>

<main>
  <svelte:component this={views[currentView]} />
</main>
