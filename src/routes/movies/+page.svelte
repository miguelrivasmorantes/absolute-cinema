<script lang="ts">
  import { page } from "$app/stores";
  import { onDestroy } from "svelte";
  import Card from "$lib/components/Card.svelte";
  import Search from "$lib/components/Search.svelte";

  type Movie = {
    id: number;
    titulo: string;
    sinopsis: string;
    posters: string;
  };

  let movies: Movie[] = [];
  let searchTerm = "";
  let perPage = 12;
  let currentPage = 1;
  let totalMovies = 0;
  let timeout: any = null;
  let orderBy = "";
  let orderDirection = "";

  async function loadMovies() {
    let url = new URL("http://127.0.0.1:8000/api/peliculas");

    if (searchTerm) url.searchParams.append("titulo", searchTerm);
    url.searchParams.append("per_page", perPage.toString());
    url.searchParams.append("page", currentPage.toString());

    if (orderBy && orderDirection) {
      url.searchParams.append(orderBy, orderDirection);
    }

    const response = await fetch(url.toString());
    const data = await response.json();
    movies = data.peliculas;
  }

  async function loadTotalMovies() {
    let url = new URL("http://127.0.0.1:8000/api/peliculas/filters-data");

    if (searchTerm) url.searchParams.append("titulo", searchTerm);

    const response = await fetch(url.toString());
    const data = await response.json();

    totalMovies = data.filters_data.peliculas_totales;
  }

  let unsubscribe = page.subscribe(($page) => {
    const params = $page.url.searchParams;

    searchTerm = params.get("titulo") || "";
    perPage = Number(params.get("per_page")) || 12;
    currentPage = Number(params.get("page")) || 1;

    loadMovies();
    loadTotalMovies();
  });

  onDestroy(() => {
    unsubscribe();
  });

  function updateFilters(newFilters: { titulo?: string; per_page?: number; page?: number; orderBy?: string; orderDirection?: string }) {
    const url = new URL(window.location.href);

    url.searchParams.delete("orden_alfabetico");
    url.searchParams.delete("orden_estreno");
    url.searchParams.delete("orden_taquilla");

    if (newFilters.titulo !== undefined) {
      newFilters.titulo
        ? url.searchParams.set("titulo", newFilters.titulo)
        : url.searchParams.delete("titulo");
    }

    if (newFilters.per_page !== undefined) {
      url.searchParams.set("per_page", newFilters.per_page.toString());
    }

    if (newFilters.page !== undefined) {
      url.searchParams.set("page", newFilters.page.toString());
    }

    if (newFilters.orderBy && newFilters.orderDirection) {
      url.searchParams.set(newFilters.orderBy, newFilters.orderDirection);
    }

    window.history.replaceState({}, "", url.toString());
    loadMovies();
  }



  function handleSearch(inputValue: string) {
    if (timeout) {
      clearTimeout(timeout);
    }

    timeout = setTimeout(() => {
      searchTerm = inputValue;
      updateFilters({ titulo: inputValue, page: 1 });
    }, 400);
  }

  function handlePerPageChange(event: Event) {
    const newPerPage = Number((event.target as HTMLSelectElement).value);
    perPage = newPerPage;
    updateFilters({ per_page: newPerPage, page: 1 });
  }

  function goToPage(pageNumber: number) {
    if (pageNumber < 1 || pageNumber > Math.ceil(totalMovies / perPage)) return;
    currentPage = pageNumber;
    updateFilters({ page: pageNumber });
    window.scrollTo({ top: 0, behavior: "smooth" });
  }

  function handleOrderChange(event: Event) {
    const selectedValue = (event.target as HTMLSelectElement).value;

    orderBy = "";
    orderDirection = "";

    if (selectedValue.includes("orden_alfabetico")) {
      const [newOrderBy, newOrderDirection] = selectedValue.split("=");
      orderBy = newOrderBy;
      orderDirection = newOrderDirection;
    } else if (selectedValue.includes("orden_estreno")) {
      const [newOrderBy, newOrderDirection] = selectedValue.split("=");
      orderBy = newOrderBy;
      orderDirection = newOrderDirection;
    } else if (selectedValue.includes("orden_taquilla")) {
      const [newOrderBy, newOrderDirection] = selectedValue.split("=");
      orderBy = newOrderBy;
      orderDirection = newOrderDirection;
    }

    currentPage = 1;
    updateFilters({ orderBy, orderDirection, page: 1 });
  }


</script>

<section class="space-y-4 mx-20">
  <h1 class="text-3xl font-bold">Movies</h1>

  <div class="flex items-center gap-4">
    <Search bind:searchTerm onSearch={handleSearch} placeholder={"Search Movies"} />

    <select class="border p-2 rounded" on:change={handleOrderChange}>
      <option value="anadidas_recientemente">Añadidas Recientemente</option>
      <option value="orden_alfabetico=asc">Título (A-Z)</option>
      <option value="orden_alfabetico=desc">Título (Z-A)</option>
      <option value="orden_estreno=asc">Estreno (Más antiguo primero)</option>
      <option value="orden_estreno=desc">Estreno (Más reciente primero)</option>
      <option value="orden_taquilla=asc">Taquilla (Menor recaudación primero)</option>
      <option value="orden_taquilla=desc">Taquilla (Mayor recaudación primero)</option>
    </select>

    <select class="border p-2 rounded" bind:value={perPage} on:change={handlePerPageChange}>
      <option value={6}>6</option>
      <option value={12}>12</option>
      <option value={24}>24</option>
      <option value={48}>48</option>
    </select>
    
  </div>
  
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {#each movies as movie}
    <Card title={movie.titulo} description={movie.sinopsis} image={movie.posters} />
    {/each}
  </div>

  {#if totalMovies > perPage}
    <div class="flex justify-center mt-4 gap-2">
      <button on:click={() => goToPage(currentPage - 1)} disabled={currentPage === 1} class="px-3 py-2 border rounded disabled:opacity-50">
        ← Anterior
      </button>
      
      {#each Array(Math.ceil(totalMovies / perPage)).fill(0) as _, i}
        <button
          class="px-3 py-2 border rounded disabled:opacity-50"
          class:disabled={currentPage === i + 1}
          disabled={currentPage === i + 1}
          on:click={() => goToPage(i + 1)}
        >
          {i + 1}
        </button>
      {/each}

      <button on:click={() => goToPage(currentPage + 1)} 
        disabled={currentPage >= Math.ceil(totalMovies / perPage)} 
        class="px-3 py-2 border rounded disabled:opacity-50">
        Siguiente →
      </button>
    </div>
  {/if}
</section>
