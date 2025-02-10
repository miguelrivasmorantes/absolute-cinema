<script>
    // Puedes importar un logo o imagen aquí si lo necesitas
  </script>
  
  <div class="navbar flex items-center justify-between bg-gray-900 p-[20px] px-[100px]">
    <!-- Logo -->
    <img src="/cinema_old_icon.svg" alt="Logo" class="w-[70px] filter invert-[0.9]" />
  
    <!-- Navegación -->
    <nav>
      <ul class="flex space-x-6">
        <li><a class="text-white hover:text-blue-400" href="/">Home</a></li>
        <li><a class="text-white hover:text-blue-400" href="/movies">Movies</a></li>
        <li><a class="text-white hover:text-blue-400" href="/actors">Actors</a></li>
      </ul>
    </nav>
  </div>
  
  <!-- Aquí se renderizan las páginas -->
  <slot class="bg-white" />
  