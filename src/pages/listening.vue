<template>
  <div class="space-y-5 py-8">
    <section class="rounded-xl border border-slate-200 bg-white p-5 shadow-sm dark:border-slate-800 dark:bg-slate-950 sm:p-6">
      <div class="mb-5">
        <h1 class="text-2xl font-black text-slate-950 dark:text-white">
          听力训练
        </h1>
        <p class="mt-2 text-sm text-slate-600 dark:text-slate-300">
          基础方法、考点词和语料练习集中整理。
        </p>
      </div>

      <nav class="grid gap-2 rounded-lg border border-slate-200 bg-slate-50 p-1 text-sm font-semibold dark:border-slate-800 dark:bg-slate-900 sm:grid-cols-3">
        <router-link
          to="/listening"
          class="rounded-md px-4 py-3 text-center text-slate-600 transition hover:bg-white hover:text-slate-950 dark:text-slate-300 dark:hover:bg-slate-800 dark:hover:text-white"
          :class="{ 'bg-white text-slate-950 shadow-sm dark:bg-slate-800 dark:text-white': $route.path === '/listening' }"
        >
          听力基础概念
        </router-link>
        <router-link
          to="/listening/keyword"
          class="rounded-md px-4 py-3 text-center text-slate-600 transition hover:bg-white hover:text-slate-950 dark:text-slate-300 dark:hover:bg-slate-800 dark:hover:text-white"
          :class="{ 'bg-white text-slate-950 shadow-sm dark:bg-slate-800 dark:text-white': $route.path === '/listening/keyword' }"
        >
          听力 179 考点词
        </router-link>
        <router-link
          to="/listening/corpus"
          class="rounded-md px-4 py-3 text-center text-slate-600 transition hover:bg-white hover:text-slate-950 dark:text-slate-300 dark:hover:bg-slate-800 dark:hover:text-white"
          :class="{ 'bg-white text-slate-950 shadow-sm dark:bg-slate-800 dark:text-white': $route.path === '/listening/corpus' }"
        >
          听力语料练习
        </router-link>
      </nav>

      <router-view />
    </section>
  </div>
</template>
