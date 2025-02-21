<script lang="ts">
  import { page } from "$app/stores";
  import { onDestroy, onMount } from "svelte";
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
  let selectedGenres: string[] = [];
  let allGenres: string[] = [];
  let genreSearchTerm = "";
  let showGenreDropdown = false;
  let availableGenres: string[] = [];
  let selectedActors: string[] = [];
  let allActors: string[] = [];
  let actorSearchTerm = "";
  let showActorDropdown = false;
  let availableActors: string[] = [];
  let selectedDirector = "";
  let selectedCountry = "";
  let allDirectors: string[] = [];
  let allCountries: string[] = [];
  let directorSearchTerm = "";
  let countrySearchTerm = "";
  let showDirectorDropdown = false;
  let showCountryDropdown = false;

  async function loadMovies() {
    let url = new URL("http://127.0.0.1:8000/api/peliculas");

    if (searchTerm) url.searchParams.append("titulo", searchTerm);
    url.searchParams.append("per_page", perPage.toString());
    url.searchParams.append("page", currentPage.toString());

    if (orderBy && orderDirection) {
      url.searchParams.append(orderBy, orderDirection);
    }

    if (selectedGenres.length > 0) {
      selectedGenres.forEach((genre) =>
        url.searchParams.append("generos[]", genre)
      );
    }

    if (selectedActors.length > 0) {
      selectedActors.forEach((actor) =>
        url.searchParams.append("actores[]", actor)
      );
    }

    if (selectedDirector) {
      url.searchParams.append("director", selectedDirector);
    }

    if (selectedCountry) {
      url.searchParams.append("pais", selectedCountry);
    }

    const response = await fetch(url.toString());
    const data = await response.json();
    movies = data.peliculas;
    totalMovies = data.peliculas_totales;
  }

  async function loadFiltersData() {
    let url = new URL("http://127.0.0.1:8000/api/peliculas/filters-data");
    const response = await fetch(url.toString());
    const data = await response.json();

    allGenres = data.filters_data.generos;
    availableGenres = [...allGenres];

    allActors = data.filters_data.actores;
    availableActors = [...allActors];

    allDirectors = data.filters_data.directores;
    allCountries = data.filters_data.paises;
  }

  let unsubscribe = page.subscribe(($page) => {
    const params = $page.url.searchParams;

    searchTerm = params.get("titulo") || "";
    perPage = Number(params.get("per_page")) || 12;
    currentPage = Number(params.get("page")) || 1;

    selectedGenres = params.getAll("generos[]") || [];
    selectedActors = params.getAll("actores[]") || [];

    selectedDirector = params.get("director") || "";
    selectedCountry = params.get("pais") || "";

    loadMovies();
    loadFiltersData();
  });

  onMount(() => {
    loadFiltersData();
  });

  onDestroy(() => {
    unsubscribe();
  });

  function updateFilters(newFilters: {
    titulo?: string;
    per_page?: number;
    page?: number;
    orderBy?: string;
    orderDirection?: string;
    generos?: string[];
    actores?: string[];
    director?: string;
    pais?: string;
  }) {
    const url = new URL(window.location.href);

    url.searchParams.delete("orden_alfabetico");
    url.searchParams.delete("orden_estreno");
    url.searchParams.delete("orden_taquilla");
    url.searchParams.delete("generos[]");
    url.searchParams.delete("actores[]");
    url.searchParams.delete("director");
    url.searchParams.delete("pais");

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

    if (newFilters.generos && newFilters.generos.length > 0) {
      newFilters.generos.forEach((genre) =>
        url.searchParams.append("generos[]", genre)
      );
    }

    if (newFilters.actores && newFilters.actores.length > 0) {
      newFilters.actores.forEach((actor) =>
        url.searchParams.append("actores[]", actor)
      );
    }

    if (newFilters.director !== undefined) {
      selectedDirector = newFilters.director;
    }

    if (newFilters.pais !== undefined) {
      selectedCountry = newFilters.pais;
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

    if (selectedValue.includes("orden_")) {
      const [newOrderBy, newOrderDirection] = selectedValue.split("=");
      orderBy = newOrderBy;
      orderDirection = newOrderDirection;
    }

    currentPage = 1;
    updateFilters({ orderBy, orderDirection, page: 1 });
  }

  function selectGenre(genre: string) {
    if (!selectedGenres.includes(genre)) {
      selectedGenres = [...selectedGenres, genre];
      availableGenres = availableGenres.filter((g) => g !== genre); // 🔹 Lo elimina del dropdown
    }

    showGenreDropdown = false;
    updateFilters({ page: 1 });
  }

  function removeGenre(genre: string) {
    selectedGenres = selectedGenres.filter((g) => g !== genre);
    availableGenres = [...availableGenres, genre].sort(); // 🔹 Lo vuelve a agregar ordenado

    updateFilters({ page: 1 });
  }

  onMount(() => {
    function handleClick(event: MouseEvent) {
      const dropdown = document.getElementById("genre-dropdown");
      const button = document.getElementById("genre-button");

      if (
        dropdown &&
        button &&
        !dropdown.contains(event.target as Node) &&
        !button.contains(event.target as Node)
      ) {
        showGenreDropdown = false;
      }
    }

    document.addEventListener("click", handleClick);

    onDestroy(() => {
      document.removeEventListener("click", handleClick);
    });
  });

  function selectActor(actor: string) {
    if (!selectedActors.includes(actor)) {
      selectedActors = [...selectedActors, actor];
      availableActors = availableActors.filter((a) => a !== actor);
    }

    showActorDropdown = false;
    updateFilters({ page: 1 });
  }

  function removeActor(actor: string) {
    selectedActors = selectedActors.filter((a) => a !== actor);
    availableActors = [...availableActors, actor].sort();

    updateFilters({ page: 1 });
  }

  function selectDirector(director: string) {
    selectedDirector = director;
    showDirectorDropdown = false; // 🔹 Cierra el dropdown
    directorSearchTerm = ""; // 🔹 Limpia el input de búsqueda
    updateFilters({ director });
  }

  function selectCountry(country: string) {
    selectedCountry = country;
    showCountryDropdown = false; // 🔹 Cierra el dropdown
    countrySearchTerm = ""; // 🔹 Limpia el input de búsqueda
    updateFilters({ pais: country });
  }
</script>

<section class="space-y-4 mx-20">
  <h1 class="text-3xl font-bold mt-3">Movies</h1>

  <div class="flex items-center gap-4">
    <Search
      bind:searchTerm
      onSearch={handleSearch}
      placeholder={"Search Movies"}
    />

    <select class="border p-2 rounded" on:change={handleOrderChange}>
      <option value="anadidas_recientemente">Añadidas Recientemente</option>
      <option value="orden_alfabetico=asc">Título (A-Z)</option>
      <option value="orden_alfabetico=desc">Título (Z-A)</option>
      <option value="orden_estreno=asc">Estreno (Más antiguo primero)</option>
      <option value="orden_estreno=desc">Estreno (Más reciente primero)</option>
      <option value="orden_taquilla=asc"
        >Taquilla (Menor recaudación primero)</option
      >
      <option value="orden_taquilla=desc"
        >Taquilla (Mayor recaudación primero)</option
      >
    </select>

    <select
      class="border p-2 rounded"
      bind:value={perPage}
      on:change={handlePerPageChange}
    >
      <option value={6}>6</option>
      <option value={12}>12</option>
      <option value={24}>24</option>
      <option value={48}>48</option>
    </select>

    <div class="relative">
      <button
        id="genre-button"
        class="border p-2 rounded bg-gray-100 hover:bg-gray-200"
        on:click={() => (showGenreDropdown = !showGenreDropdown)}
      >
        Géneros
      </button>

      {#if showGenreDropdown}
        <div
          id="genre-dropdown"
          class="absolute top-full left-0 w-48 bg-white border rounded shadow-lg z-50 max-h-60 overflow-auto mt-2"
        >
          <div class="sticky top-0 bg-white p-2 border-b">
            <input
              type="text"
              bind:value={genreSearchTerm}
              placeholder="Buscar género..."
              class="border p-2 rounded w-full"
            />
          </div>

          {#each availableGenres.filter((g) => g
              .toLowerCase()
              .includes(genreSearchTerm.toLowerCase())) as genre}
            <div
              class="p-2 hover:bg-gray-200 cursor-pointer"
              on:click={() => selectGenre(genre)}
            >
              {genre}
            </div>
          {/each}
        </div>
      {/if}
    </div>

    <div class="relative">
      <button
        id="actor-button"
        class="border p-2 rounded bg-gray-100 hover:bg-gray-200"
        on:click={() => (showActorDropdown = !showActorDropdown)}
      >
        Actores
      </button>

      {#if showActorDropdown}
        <div
          id="actor-dropdown"
          class="absolute top-full left-0 w-48 bg-white border rounded shadow-lg z-50 max-h-60 overflow-auto mt-2"
        >
          <div class="sticky top-0 bg-white p-2 border-b">
            <input
              type="text"
              bind:value={actorSearchTerm}
              placeholder="Buscar actor..."
              class="border p-2 rounded w-full"
            />
          </div>

          {#each availableActors.filter((a) => a
              .toLowerCase()
              .includes(actorSearchTerm.toLowerCase())) as actor}
            <div
              class="p-2 hover:bg-gray-200 cursor-pointer"
              on:click={() => selectActor(actor)}
            >
              {actor}
            </div>
          {/each}
        </div>
      {/if}
    </div>

    <div class="relative">
      <button
        id="director-button"
        class="border p-2 rounded bg-gray-100 hover:bg-gray-200"
        on:click={() => (showDirectorDropdown = !showDirectorDropdown)}
      >
        {selectedDirector ? selectedDirector : "Director"}
      </button>

      {#if showDirectorDropdown}
        <div
          id="director-dropdown"
          class="absolute top-full left-0 w-48 bg-white border rounded shadow-lg z-50 max-h-60 overflow-auto mt-2"
        >
          <div class="sticky top-0 bg-white p-2 border-b">
            <input
              type="text"
              bind:value={directorSearchTerm}
              placeholder="Buscar director..."
              class="border p-2 rounded w-full"
            />
          </div>

          {#each allDirectors.filter((d) => d
              .toLowerCase()
              .includes(directorSearchTerm.toLowerCase())) as director}
            <div
              class="p-2 hover:bg-gray-200 cursor-pointer"
              on:click={() => selectDirector(director)}
            >
              {director}
            </div>
          {/each}
        </div>
      {/if}
    </div>

    <div class="relative">
      <button
        id="country-button"
        class="border p-2 rounded bg-gray-100 hover:bg-gray-200"
        on:click={() => (showCountryDropdown = !showCountryDropdown)}
      >
        {selectedCountry ? selectedCountry : "País"}
      </button>

      {#if showCountryDropdown}
        <div
          id="country-dropdown"
          class="absolute top-full left-0 w-48 bg-white border rounded shadow-lg z-50 max-h-60 overflow-auto mt-2"
        >
          <div class="sticky top-0 bg-white p-2 border-b">
            <input
              type="text"
              bind:value={countrySearchTerm}
              placeholder="Buscar país..."
              class="border p-2 rounded w-full"
            />
          </div>

          {#each allCountries.filter((c) => c
              .toLowerCase()
              .includes(countrySearchTerm.toLowerCase())) as country}
            <div
              class="p-2 hover:bg-gray-200 cursor-pointer"
              on:click={() => selectCountry(country)}
            >
              {country}
            </div>
          {/each}
        </div>
      {/if}
    </div>
  </div>

  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {#each movies as movie}
      <Card
        title={movie.titulo}
        description={movie.sinopsis}
        image={movie.posters}
      />
    {/each}
  </div>

  {#if totalMovies > perPage}
    <div class="flex justify-center mt-4 gap-2">
      <button
        on:click={() => goToPage(currentPage - 1)}
        disabled={currentPage === 1}
        class="px-3 py-2 border rounded disabled:opacity-50"
      >
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

      <button
        on:click={() => goToPage(currentPage + 1)}
        disabled={currentPage >= Math.ceil(totalMovies / perPage)}
        class="px-3 py-2 border rounded disabled:opacity-50"
      >
        Siguiente →
      </button>
    </div>
  {/if}
</section>
