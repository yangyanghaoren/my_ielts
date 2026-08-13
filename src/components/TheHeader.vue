<script setup lang="ts" generic="T extends any, O extends any">
const menus = reactive([
  {
    label: '首页',
    icon: 'i-carbon-home',
    link: '/',
  },
  {
    label: '词汇',
    icon: 'i-carbon-chart-histogram',
    link: '/vocabulary',
  },
  {
    label: '语法',
    icon: 'i-carbon-load-balancer-vpc',
    link: '/grammar',
  },
  {
    label: '听力',
    icon: 'i-carbon-headphones',
    link: '/listening',
  },
  {
    label: '口语',
    icon: 'i-carbon-microphone',
    link: '/speaking',
  },
  {
    label: '阅读',
    icon: 'i-carbon-white-paper',
    link: '/reading',
  },
  {
    label: '写作',
    icon: 'i-carbon-edit',
    link: '/writing',
  },
])

const showMobileMenu = ref(false)
</script>

<template>
  <header class="fixed inset-x-0 top-0 z-30 border-b border-slate-200/70 bg-white/88 backdrop-blur-xl dark:border-slate-800 dark:bg-slate-950/86">
    <nav class="mx-auto max-w-screen-2xl px-4 sm:px-6 lg:px-8">
      <div class="flex h-16 items-center justify-between gap-4">
        <div class="flex min-w-0 items-center gap-8">
          <router-link to="/" class="flex items-center gap-3" @click="showMobileMenu = false">
            <span class="grid h-10 w-10 place-items-center rounded-lg bg-slate-950 text-white shadow-sm dark:bg-white dark:text-slate-950">
              <i class="i-carbon-education text-xl" />
            </span>
            <span class="hidden min-w-0 sm:block">
              <span class="block whitespace-nowrap text-base font-bold leading-5 text-slate-950 dark:text-white">English Prep Lab</span>
              <span class="block whitespace-nowrap text-xs leading-4 text-slate-500 dark:text-slate-400">Practice workspace</span>
            </span>
          </router-link>

          <ul class="hidden items-center gap-1 rounded-lg border border-slate-200 bg-slate-50 p-1 text-sm font-medium dark:border-slate-800 dark:bg-slate-900 lg:flex">
            <li v-for="m in menus" :key="m.label">
              <router-link
                :to="m.link"
                class="flex items-center gap-1.5 rounded-md px-3 py-2 text-slate-600 transition hover:bg-white hover:text-slate-950 dark:text-slate-300 dark:hover:bg-slate-800 dark:hover:text-white"
                :class="{ 'bg-white text-slate-950 shadow-sm dark:bg-slate-800 dark:text-white': $route.path === m.link }"
              >
                <i class="inline-block" :class="m.icon" />
                {{ m.label }}
              </router-link>
            </li>
          </ul>
        </div>

        <div class="flex items-center gap-2">
          <a
            href="https://github.com/yangyanghaoren/my_ielts"
            target="_blank"
            class="grid h-10 w-10 place-items-center rounded-lg border border-slate-200 bg-white text-slate-600 transition hover:border-slate-300 hover:text-slate-950 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-300 dark:hover:text-white"
            title="GitHub"
          >
            <div i-simple-icons-github />
          </a>
          <button
            class="grid h-10 w-10 place-items-center rounded-lg border border-slate-200 bg-white text-slate-600 transition hover:border-slate-300 hover:text-slate-950 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-300 dark:hover:text-white"
            title="切换深色模式"
            @click="toggleDark()"
          >
            <div i-carbon-sun dark:i-carbon-moon />
          </button>

          <button
            type="button"
            class="grid h-10 w-10 place-items-center rounded-lg border border-slate-200 bg-white text-slate-600 transition hover:border-slate-300 hover:text-slate-950 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-300 dark:hover:text-white lg:hidden"
            aria-label="打开导航"
            @click="showMobileMenu = !showMobileMenu"
          >
            <i :class="showMobileMenu ? 'i-carbon-close' : 'i-carbon-menu'" />
          </button>
        </div>
      </div>

      <ul
        v-show="showMobileMenu"
        class="grid gap-1 border-t border-slate-200 py-3 text-sm font-medium dark:border-slate-800 lg:hidden"
      >
        <li v-for="m in menus" :key="m.label">
          <router-link
            :to="m.link"
            class="flex items-center gap-2 rounded-lg px-3 py-2.5 text-slate-700 hover:bg-slate-100 dark:text-slate-200 dark:hover:bg-slate-900"
            :class="{ 'bg-slate-100 text-slate-950 dark:bg-slate-900 dark:text-white': $route.path === m.link }"
            @click="showMobileMenu = false"
          >
            <i class="inline-block" :class="m.icon" />
            {{ m.label }}
          </router-link>
        </li>
      </ul>
    </nav>
  </header>
</template>
