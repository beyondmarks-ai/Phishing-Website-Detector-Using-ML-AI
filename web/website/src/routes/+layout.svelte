<script>
  import "../app.css";
  const year = new Date().getFullYear();
</script>

<div class="min-h-screen flex flex-col">
  <main class="flex-1">
    <slot />
  </main>

  <footer class="bg-gray-900 text-gray-300 border-t border-gray-800 py-6">
    <div class="max-w-4xl mx-auto px-6 text-sm md:text-base text-center text-gray-400">
      © {year} Beyond Marks AI Academy
    </div>
  </footer>
</div>
