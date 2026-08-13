<script setup lang="ts" generic="T extends any, O extends any">
import { RouterLink } from 'vue-router'

defineOptions({
  name: 'IndexPage',
})

const menus = reactive([
  {
    label: '主题词汇',
    icon: 'i-carbon-chart-histogram',
    link: '/vocabulary',
    desc: '搜索、筛选、音频和听写模式集中在一个页面。',
    meta: '2660+ words',
    tone: 'from-cyan-500 to-blue-600',
  },
  {
    label: '语法复习',
    icon: 'i-carbon-load-balancer-vpc',
    link: '/grammar',
    desc: '基础结构、讲义和图谱资料，适合查漏补缺。',
    meta: 'Notes',
    tone: 'from-emerald-500 to-teal-600',
  },
  {
    label: '听力训练',
    icon: 'i-carbon-headphones',
    link: '/listening',
    desc: '听力概念、考点词和同义替换练习入口。',
    meta: 'Audio',
    tone: 'from-amber-500 to-orange-600',
  },
  {
    label: '阅读表达',
    icon: 'i-carbon-white-paper',
    link: '/reading',
    desc: '阅读高频考点词和同义表达整理。',
    meta: '538 keys',
    tone: 'from-violet-500 to-fuchsia-600',
  },
  {
    label: '写作句型',
    icon: 'i-carbon-edit',
    link: '/writing',
    desc: '翻译练习、句型积累和大小作文复习入口。',
    meta: '100 lines',
    tone: 'from-rose-500 to-red-600',
  },
  {
    label: '单词跟打',
    icon: 'i-carbon-keyboard',
    link: '/vocabulary/typing',
    desc: '用输入练习加强拼写记忆和反应速度。',
    meta: 'Typing',
    tone: 'from-slate-600 to-slate-900',
  },
])

const stats = [
  { label: '学习模块', value: '6' },
  { label: '词汇分类', value: '22' },
  { label: '练习模式', value: '3' },
]
</script>

<template>
  <div class="space-y-8 py-8">
    <section class="overflow-hidden rounded-xl border border-slate-200 bg-white shadow-sm dark:border-slate-800 dark:bg-slate-950">
      <div class="grid gap-0 lg:grid-cols-[1.12fr_0.88fr]">
        <div class="p-6 sm:p-8 lg:p-10">
          <div class="mb-5 inline-flex items-center gap-2 rounded-lg border border-slate-200 bg-slate-50 px-3 py-1.5 text-sm font-medium text-slate-600 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-300">
            <i class="i-carbon-education" />
            Personal English Practice Workspace
          </div>
          <h1 class="max-w-3xl text-4xl font-black leading-tight text-slate-950 dark:text-white sm:text-5xl lg:text-6xl">
            English Prep Lab
          </h1>
          <p class="mt-5 max-w-2xl text-base leading-8 text-slate-600 dark:text-slate-300 sm:text-lg">
            把词汇、听力、阅读、写作和语法复习放在一个安静的练习台里。打开就能查词、筛选、听音频、做听写，不再在一堆资料之间来回跳。
          </p>
          <div class="mt-8 flex flex-wrap gap-3">
            <RouterLink
              to="/vocabulary"
              class="inline-flex items-center gap-2 rounded-lg bg-slate-950 px-5 py-3 text-sm font-semibold text-white shadow-sm transition hover:bg-slate-800 dark:bg-white dark:text-slate-950 dark:hover:bg-slate-200"
            >
              <i class="i-carbon-search" />
              开始练词
            </RouterLink>
            <RouterLink
              to="/vocabulary/typing"
              class="inline-flex items-center gap-2 rounded-lg border border-slate-200 bg-white px-5 py-3 text-sm font-semibold text-slate-700 transition hover:border-slate-300 hover:text-slate-950 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-200 dark:hover:text-white"
            >
              <i class="i-carbon-keyboard" />
              跟打练习
            </RouterLink>
          </div>
        </div>

        <div class="border-t border-slate-200 bg-slate-50 p-6 dark:border-slate-800 dark:bg-slate-900/60 lg:border-l lg:border-t-0 sm:p-8 lg:p-10">
          <div class="grid gap-3 sm:grid-cols-3 lg:grid-cols-1">
            <div
              v-for="item in stats"
              :key="item.label"
              class="rounded-lg border border-slate-200 bg-white p-5 dark:border-slate-800 dark:bg-slate-950"
            >
              <div class="text-3xl font-black text-slate-950 dark:text-white">
                {{ item.value }}
              </div>
              <div class="mt-1 text-sm text-slate-500 dark:text-slate-400">
                {{ item.label }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section class="grid gap-4 md:grid-cols-2 xl:grid-cols-3">
      <RouterLink
        v-for="m in menus"
        :key="m.label"
        :to="m.link"
        class="group overflow-hidden rounded-xl border border-slate-200 bg-white p-5 shadow-sm transition hover:-translate-y-0.5 hover:border-slate-300 hover:shadow-md dark:border-slate-800 dark:bg-slate-950 dark:hover:border-slate-700"
      >
        <div class="mb-5 flex items-start justify-between gap-4">
          <div :class="`grid h-12 w-12 place-items-center rounded-lg bg-gradient-to-br ${m.tone} text-white shadow-sm`">
            <i class="text-xl" :class="m.icon" />
          </div>
          <span class="rounded-md bg-slate-100 px-2.5 py-1 text-xs font-semibold text-slate-500 dark:bg-slate-900 dark:text-slate-400">{{ m.meta }}</span>
        </div>
        <h2 class="text-xl font-bold text-slate-950 dark:text-white">
          {{ m.label }}
        </h2>
        <p class="mt-2 text-sm leading-7 text-slate-600 dark:text-slate-300">
          {{ m.desc }}
        </p>
        <div class="mt-5 inline-flex items-center gap-2 text-sm font-semibold text-slate-950 dark:text-white">
          进入模块
          <i class="i-carbon-arrow-right transition group-hover:translate-x-1" />
        </div>
      </RouterLink>
    </section>
  </div>
</template>
