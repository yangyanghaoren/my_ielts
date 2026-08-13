<template>
  <TheHeader />
  <div class="app-shell flex flex-1 overflow-hidden pt-18 pb-10 leading-8">
    <div class="relative mx-auto h-full w-full max-w-screen-2xl overflow-y-auto px-4 text-slate-900 dark:text-slate-100 sm:px-6 lg:px-8">
      <RouterView />
    </div>
  </div>
  <TheFooter />
</template>
