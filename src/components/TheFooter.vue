<script setup>
const year = (new Date()).getFullYear()
</script>

<template>
  <footer class="border-t border-slate-200/80 bg-white/75 px-4 py-6 backdrop-blur dark:border-slate-800 dark:bg-slate-950/75">
    <p class="mx-auto max-w-screen-2xl text-center text-sm text-slate-500 dark:text-slate-400">
      &copy; {{ year }} English Prep Lab. Built for focused English practice.
    </p>
  </footer>
</template>
