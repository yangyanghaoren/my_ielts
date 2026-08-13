<script setup>
import sentences from './100sentences'
</script>

<template>
  <div class="space-y-5 py-8">
    <section class="rounded-xl border border-slate-200 bg-white p-5 shadow-sm dark:border-slate-800 dark:bg-slate-950 sm:p-6">
      <div class="mb-5">
        <h1 class="text-2xl font-black text-slate-950 dark:text-white">
          写作训练
        </h1>
        <p class="mt-2 text-sm text-slate-600 dark:text-slate-300">
          从句子翻译开始，把表达准确度和结构感练稳。
        </p>
      </div>

      <nav class="mb-6 grid gap-2 rounded-lg border border-slate-200 bg-slate-50 p-1 text-sm font-semibold dark:border-slate-800 dark:bg-slate-900 sm:grid-cols-3">
        <a href="/#/writing" class="rounded-md bg-white px-4 py-3 text-center text-slate-950 shadow-sm dark:bg-slate-800 dark:text-white">100 句翻译练习</a>
        <a href="/#/writing" class="rounded-md px-4 py-3 text-center text-slate-600 transition hover:bg-white hover:text-slate-950 dark:text-slate-300 dark:hover:bg-slate-800 dark:hover:text-white">[WIP] 议论文训练</a>
        <a href="/#/writing" class="rounded-md px-4 py-3 text-center text-slate-600 transition hover:bg-white hover:text-slate-950 dark:text-slate-300 dark:hover:bg-slate-800 dark:hover:text-white">[WIP] 图表写作</a>
      </nav>

      <div class="overflow-hidden rounded-xl border border-slate-200 dark:border-slate-800">
        <table class="min-w-full divide-y divide-slate-200 dark:divide-slate-800">
          <tbody class="divide-y divide-slate-100 bg-white dark:divide-slate-800 dark:bg-slate-950">
            <template v-for="item of sentences" :key="item.no">
              <template v-if="item.no == null">
                <tr class="bg-slate-50 dark:bg-slate-900">
                  <td colspan="5" class="px-4 py-4 text-lg font-bold text-slate-950 dark:text-white">
                    {{ item.title }}
                  </td>
                </tr>
              </template>
              <template v-else>
                <tr class="align-top text-sm text-slate-700 dark:text-slate-200">
                  <td class="w-16 border-r border-slate-100 px-4 py-4 font-mono text-xs text-slate-500 dark:border-slate-800 dark:text-slate-400" rowspan="2">
                    {{ item.no }}
                  </td>
                  <td class="min-w-80 border-r border-slate-100 px-4 py-4 font-medium text-slate-950 dark:border-slate-800 dark:text-white" rowspan="2">
                    {{ item.sentence }}
                  </td>
                  <td class="border-b border-slate-100 px-4 py-4 dark:border-slate-800">
                    <div class="mb-2 inline-flex items-center gap-2 rounded-md bg-slate-100 px-2 py-1 text-xs font-semibold text-slate-500 dark:bg-slate-900 dark:text-slate-400">
                      <i class="i-carbon-book" />
                      参考译文
                    </div>
                    <div>{{ item.translationFromBook }}</div>
                  </td>
                  <td class="w-30% border-l border-slate-100 px-4 py-4 whitespace-pre-line text-slate-500 dark:border-slate-800 dark:text-slate-400" rowspan="2">
                    {{ item.remark }}
                  </td>
                </tr>
                <tr class="align-top text-sm text-slate-700 dark:text-slate-200">
                  <td class="px-4 py-4">
                    <div class="mb-2 inline-flex items-center gap-2 rounded-md bg-slate-100 px-2 py-1 text-xs font-semibold text-slate-500 dark:bg-slate-900 dark:text-slate-400">
                      <i class="i-carbon-chat" />
                      改写参考
                    </div>
                    <div>{{ item.chatgpt }}</div>
                  </td>
                </tr>
              </template>
            </template>
          </tbody>
        </table>
      </div>
    </section>
  </div>
</template>
