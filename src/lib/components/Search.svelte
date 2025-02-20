<script>
  export let searchTerm = "";
  export let placeholder = "";
  export let onSearch = () => {};
</script>

<input
  type="text"
  placeholder={placeholder}
  bind:value={searchTerm}
  on:input={() => onSearch(searchTerm)}
  class="search-bar"
  autofocus
/>

<style>
  .search-bar {
    width: 100%;
    padding: 0.5rem;
    border-radius: 5px;
    border: 1px solid #ccc;
    padding: 8px;
    font-size: 1rem;
  }
</style>
