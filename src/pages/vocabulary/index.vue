<!-- eslint-disable eslint-comments/no-unlimited-disable -->
<script setup generic="T extends any, O extends any">
import vocabulary from './vocabulary'
import { getCategoryLabel } from './categoryLabels'

const CHAPTER_KEY = 'vocabulary_chapter'
const EXTRA_OVERRIDES_KEY = 'vocabulary_extra_overrides'
const DEFAULT_SCOPE_KEY = 'vocabulary_default_scope_v2'

function loadExtraOverrides() {
  try {
    return JSON.parse(localStorage.getItem(EXTRA_OVERRIDES_KEY) || '{}')
  }
  catch {
    return {}
  }
}

function saveExtraOverride(id, value) {
  const overrides = loadExtraOverrides()
  overrides[id] = value
  localStorage.setItem(EXTRA_OVERRIDES_KEY, JSON.stringify(overrides))
}

function onExtraBlur(e, item) {
  const value = e.target.textContent.trim()
  item.extra = value
  saveExtraOverride(item.id, value)
}

const isTrainingModel = ref(false)
const isShowMeaning = ref(true)
const isAutoPlayWordAudio = ref(true)
const isOnlyShowErrors = ref(false)
const isFinishTraining = ref(false)
const isShowSource = ref(false)

const trainingStats = ref('')
const keyword = ref('')
const ALL_CATEGORY = '__all__'
const chapters = Object.keys(vocabulary)
const savedCategory = localStorage.getItem(CHAPTER_KEY)
const hasDefaultScope = localStorage.getItem(DEFAULT_SCOPE_KEY) === '1'
const initialCategory = !hasDefaultScope
  ? ALL_CATEGORY
  : savedCategory === ALL_CATEGORY || chapters.includes(savedCategory) ? savedCategory : ALL_CATEGORY
const category = ref(initialCategory)
localStorage.setItem(DEFAULT_SCOPE_KEY, '1')
if (!hasDefaultScope)
  localStorage.setItem(CHAPTER_KEY, ALL_CATEGORY)

const priorityFilter = ref('all')
const FREQUENCY_LABELS = { high: '高频', medium: '中频', low: '低频' }
const FREQUENCY_CLASSES = {
  high: 'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-300',
  medium: 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-300',
  low: 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300',
}
const MASTERY_LABELS = { productive: '产出', receptive: '识记' }
const MASTERY_CLASSES = {
  productive: 'bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-300',
  receptive: 'bg-purple-100 text-purple-800 dark:bg-purple-900 dark:text-purple-300',
}

const importanceChecks = reactive({ 1: false, 2: false, 3: false, 4: false, 5: false })
const hasImportanceSelection = computed(() => Object.values(importanceChecks).some(Boolean))

const PRIORITY_OPTIONS = [
  { value: 'all', label: '全部词汇' },
  { value: 'gte4', label: '重要度 ≥4（核心词）' },
  { value: 'gte3', label: '重要度 ≥3' },
  { value: 'gte2', label: '重要度 ≥2' },
  { value: 'high', label: '仅高频词' },
  { value: 'productive', label: '仅产出型' },
  { value: 'receptive', label: '仅识记型' },
]

const isFilterOpen = ref(false)
const filterDropdownRef = ref(null)
onClickOutside(filterDropdownRef, () => {
  isFilterOpen.value = false
})

const filterSummaryLabel = computed(() => {
  if (hasImportanceSelection.value) {
    const selected = [5, 4, 3, 2, 1].filter(n => importanceChecks[n])
    return `重要度：${selected.join('、')}`
  }
  return PRIORITY_OPTIONS.find(opt => opt.value === priorityFilter.value)?.label || '全部词汇'
})

function onImportanceCheckboxChange() {
  priorityFilter.value = 'all'
}

function onPriorityFilterChange() {
  for (const n in importanceChecks)
    importanceChecks[n] = false
}

function selectPriorityOption(value) {
  priorityFilter.value = value
  onPriorityFilterChange()
  isFilterOpen.value = false
}

function matchesPriorityFilter(item) {
  if (hasImportanceSelection.value)
    return !!importanceChecks[item.importance]

  switch (priorityFilter.value) {
    case 'gte4':
      return item.importance >= 4
    case 'gte3':
      return item.importance >= 3
    case 'gte2':
      return item.importance >= 2
    case 'high':
      return item.frequency === 'high'
    case 'productive':
      return item.mastery === 'productive'
    case 'receptive':
      return item.mastery === 'receptive'
    default:
      return true
  }
}

const refVocabulary = reactive(vocabulary)
const searchKeyword = computed(() => keyword.value.trim().toLowerCase())
const isAllCategories = computed(() => category.value === ALL_CATEGORY)
const currentChapter = computed(() => refVocabulary[category.value])
const allGroupCount = computed(() => Object.values(refVocabulary).reduce((total, chapter) => total + chapter.groupCount, 0))
const allWordCount = computed(() => Object.values(refVocabulary).reduce((total, chapter) => total + chapter.wordCount, 0))
const selectedGroupCount = computed(() => isAllCategories.value ? allGroupCount.value : currentChapter.value?.groupCount || 0)
const selectedWordCount = computed(() => isAllCategories.value ? allWordCount.value : currentChapter.value?.wordCount || 0)
const selectedCategoryLabel = computed(() => isAllCategories.value ? '00 全部分类' : getCategoryLabel(category.value))
const isPriorityFilterActive = computed(() => hasImportanceSelection.value || priorityFilter.value !== 'all')
const isFilterActive = computed(() => isPriorityFilterActive.value || !!searchKeyword.value)

const extraOverrides = loadExtraOverrides()
const wordCategoryMap = new Map()
for (const [categoryKey, cat] of Object.entries(refVocabulary)) {
  for (const group of cat.words) {
    for (const item of group) {
      wordCategoryMap.set(item.id, categoryKey)
      if (extraOverrides[item.id] !== undefined)
        item.extra = extraOverrides[item.id]
    }
  }
}

function getItemCategory(item) {
  return wordCategoryMap.get(item.id) || category.value
}

function getWordAudioPath(item) {
  return `vocabulary/audio/${getItemCategory(item)}/${item.word[0]}.mp3`
}

function isMatchedWord(item, keywordValue) {
  if (item.word.some(word => word.toLowerCase().includes(keywordValue)))
    return true

  return [item.pos, item.meaning, item.example, item.extra]
    .some(value => (value || '').toLowerCase().includes(keywordValue))
}

function matchesWordFilters(item) {
  if (!matchesPriorityFilter(item))
    return false
  if (!searchKeyword.value)
    return true
  return isMatchedWord(item, searchKeyword.value)
}

const filteredWordGroups = computed(() => {
  const chapterEntries = isAllCategories.value
    ? Object.entries(refVocabulary)
    : [[category.value, currentChapter.value]]
  if (!chapterEntries.length)
    return []

  const groups = []
  for (const [, chapter] of chapterEntries) {
    if (!chapter)
      continue
    for (const group of chapter.words) {
      const filteredGroup = group.filter(item => matchesWordFilters(item))
      if (filteredGroup.length > 0)
        groups.push(filteredGroup)
    }
  }
  return groups
})

const filteredWordCount = computed(() => {
  return filteredWordGroups.value.reduce((total, group) => total + group.length, 0)
})

const currentCount = computed(() => isFilterActive.value ? filteredWordCount.value : selectedWordCount.value)

const filteredIndexMap = computed(() => {
  const map = new Map()
  if (!isFilterActive.value)
    return map

  let idx = 0
  for (const group of filteredWordGroups.value) {
    for (const item of group)
      map.set(item.id, ++idx)
  }
  return map
})

watch(category, (newVal, oldVal) => {
  // console.log(newVal, oldVal)
  localStorage.setItem(CHAPTER_KEY, newVal)
})

function calcStats() {
  let error = 0
  let missing = 0
  let correct = 0
  if (isTrainingModel.value) {
    const chaptersToScan = isAllCategories.value ? Object.values(refVocabulary) : [refVocabulary[category.value]]
    // 遍历所有单词的属性
    for (const cur of chaptersToScan) {
      if (!cur)
        continue
      for (const group of cur.words) {
        for (const item of group) {
          if (item.spellValue) {
            if (item.spellError)
              error++
            else
              correct++
          }
          else { missing++ }
        }
      }
    }
  }
  return `${missing} 个未完成，${correct} 个正确，${error} 个错误`
}

onMounted(() => {
  // 只能同时播放一个音频
  const audioTags = document.getElementsByTagName('audio')
  for (const audio of audioTags) {
    audio.onplay = () => {
      for (const _audio of audioTags) {
        _audio.blur()
        if (audio !== _audio)
          _audio.pause()
      }
    }
  }
})

onUpdated(() => {
  // 音频再切换 SRC 之后需要调用一下 load() 不然看不到效果
  for (const el of document.getElementsByTagName('audio'))
    el.load()
})

document.addEventListener('keydown', (ev) => {
  const target = ev.target
  const isEditingText = target.tagName === 'INPUT' || target.tagName === 'TEXTAREA' || target.isContentEditable
  if (isEditingText)
    return

  // 激活的那个音频可以通过方向键进行快进/退
  if (['ArrowLeft', 'ArrowRight', ' '].includes(ev.key)) {
    ev.preventDefault()
    const audioTags = document.getElementsByTagName('audio')
    const keyMap = {
      ArrowLeft: -5,
      ArrowRight: 5,
    }
    for (const audioTag of audioTags) {
      audioTag.blur()
      if (keyMap[ev.key]) {
        const step = keyMap[ev.key]
        audioTag.currentTime = audioTag.currentTime + step
        // console.log(step, audioT ag.currentTime)
      }
      if (ev.key === ' ') {
        if (audioTag.paused)
          audioTag.play()
        else
          audioTag.pause()
      }
    }
  }
})

let audio = null
function play(audioPath) {
  if (audio) {
    audio.pause()
    audio.currentTime = 0
  }
  audio = document.createElement('audio')
  audio.src = audioPath
  audio.play()
}

function copyText(item) {
  const text = `${item.word} ${item.pos} ${item.meaning}`
  navigator.clipboard.writeText(text)
}

function onInputKeydown(e) {
  e.stopPropagation()
  const { key, target } = e
  // console.log(key, target.id)
  if (key === 'Enter') {
    // 切换到下一个 input
    document.getElementById((Number(target.id) + 1).toString())?.focus()
  }
}

function onInputFocusIn(e, audioPath) {
  if (isAutoPlayWordAudio.value)
    play(audioPath)
}

function onInputFocusOut(e, item) {
  const { target } = e
  const spellValue = target.value.toLowerCase().trim()
  if (spellValue.length < 1) {
    item.spellValue = ''
    item.spellError = false
  }
  else {
    item.spellValue = spellValue
    item.spellError = !item.word.map(v => v.toLowerCase().trim()).includes(spellValue)
  }
  trainingStats.value = calcStats()
}

function getInputStyleClass(item) {
  const cls = {
    error: 'ml-3 inline-block w-34 rounded-lg border border-red-300 bg-red-50 px-3 py-2 text-sm text-red-900 outline-none focus:border-red-500 focus:ring-2 focus:ring-red-100 dark:border-red-800 dark:bg-red-950/40 dark:text-red-200 dark:focus:ring-red-900/60',
    normal: 'ml-3 inline-block w-34 rounded-lg border border-slate-200 bg-white px-3 py-2 text-sm text-slate-900 outline-none focus:border-slate-400 focus:ring-2 focus:ring-slate-100 dark:border-slate-700 dark:bg-slate-950 dark:text-white dark:focus:border-slate-500 dark:focus:ring-slate-800',
    success: 'ml-3 inline-block w-34 rounded-lg border border-emerald-300 bg-emerald-50 px-3 py-2 text-sm text-emerald-900 outline-none focus:border-emerald-500 focus:ring-2 focus:ring-emerald-100 dark:border-emerald-800 dark:bg-emerald-950/40 dark:text-emerald-200 dark:focus:ring-emerald-900/60',
  }
  if (isFinishTraining.value) {
    if (item.spellError)
      return cls.error
    if (item.spellValue.length > 0 && !item.spellError)
      return cls.success
  }
  return cls.normal
}

function copyAllError() {
  const chaptersToScan = isAllCategories.value ? Object.values(refVocabulary) : [refVocabulary[category.value]]
  const errorWords = []
  for (const cur of chaptersToScan) {
    if (!cur)
      continue
    for (const group of cur.words) {
      for (const item of group) {
        if (item.spellError)
          errorWords.push(`${item.word} ${item.pos} ${item.meaning}`)
      }
    }
  }
  navigator.clipboard.writeText(errorWords.join('\n\n'))
}
</script>

<template>
  <div class="space-y-6 py-8">
    <section class="grid gap-5 lg:grid-cols-[1fr_auto] lg:items-end">
      <div>
        <div class="mb-3 inline-flex items-center gap-2 rounded-lg border border-slate-200 bg-white px-3 py-1.5 text-sm font-semibold text-slate-600 shadow-sm dark:border-slate-800 dark:bg-slate-950 dark:text-slate-300">
          <i class="i-carbon-chart-histogram" />
          Vocabulary Workspace
        </div>
        <h1 class="text-3xl font-black text-slate-950 dark:text-white sm:text-4xl">
          主题词汇训练
        </h1>
        <p class="mt-3 max-w-2xl text-sm leading-7 text-slate-600 dark:text-slate-300 sm:text-base">
          按学习场景整理常用词，支持搜索、筛选、音频播放、听写和例句复习。
        </p>
      </div>

      <div class="grid grid-cols-3 gap-3 sm:min-w-120">
        <div class="rounded-lg border border-slate-200 bg-white p-4 shadow-sm dark:border-slate-800 dark:bg-slate-950">
          <div class="text-2xl font-black text-slate-950 dark:text-white">
            {{ selectedGroupCount }}
          </div>
          <div class="text-xs font-medium text-slate-500 dark:text-slate-400">
            词组
          </div>
        </div>
        <div class="rounded-lg border border-slate-200 bg-white p-4 shadow-sm dark:border-slate-800 dark:bg-slate-950">
          <div class="text-2xl font-black text-slate-950 dark:text-white">
            {{ selectedWordCount }}
          </div>
          <div class="text-xs font-medium text-slate-500 dark:text-slate-400">
            单词
          </div>
        </div>
        <div class="rounded-lg border border-slate-200 bg-white p-4 shadow-sm dark:border-slate-800 dark:bg-slate-950">
          <div class="text-2xl font-black text-slate-950 dark:text-white">
            {{ currentCount }}
          </div>
          <div class="text-xs font-medium text-slate-500 dark:text-slate-400">
            当前
          </div>
        </div>
      </div>
    </section>

    <section class="rounded-xl border border-slate-200 bg-white p-4 shadow-sm dark:border-slate-800 dark:bg-slate-950 sm:p-5">
      <div class="grid gap-3 xl:grid-cols-[minmax(12rem,16rem)_14rem_minmax(16rem,1fr)_auto] xl:items-start">
        <select
          v-model="category"
          class="h-11 rounded-lg border border-slate-200 bg-slate-50 px-3 text-sm font-medium text-slate-900 outline-none transition focus:border-slate-400 focus:bg-white focus:ring-2 focus:ring-slate-100 dark:border-slate-800 dark:bg-slate-900 dark:text-white dark:focus:border-slate-600 dark:focus:ring-slate-800"
        >
          <option :value="ALL_CATEGORY">
            00 全部分类
          </option>
          <option v-for="(_, k) in refVocabulary" :key="k" :value="k">
            {{ getCategoryLabel(k) }}
          </option>
        </select>

        <div ref="filterDropdownRef" class="relative">
          <button
            type="button"
            class="h-11 w-full flex items-center justify-between rounded-lg border border-slate-200 bg-slate-50 px-3 text-left text-sm font-medium text-slate-900 outline-none transition hover:bg-white dark:border-slate-800 dark:bg-slate-900 dark:text-white dark:hover:bg-slate-800"
            aria-haspopup="true"
            :aria-expanded="isFilterOpen"
            @click="isFilterOpen = !isFilterOpen"
          >
            <span class="truncate">{{ filterSummaryLabel }}</span>
            <i class="i-ph-caret-down-bold ml-2 flex-shrink-0 text-slate-500" />
          </button>
          <div
            v-show="isFilterOpen"
            class="absolute z-10 mt-2 w-72 rounded-lg border border-slate-200 bg-white p-2 shadow-xl dark:border-slate-800 dark:bg-slate-950"
          >
            <div class="mb-1 px-2 text-xs font-semibold uppercase tracking-wide text-slate-400 dark:text-slate-500">
              条件筛选
            </div>
            <button
              v-for="opt in PRIORITY_OPTIONS"
              :key="opt.value"
              type="button"
              class="block w-full rounded-md px-2 py-2 text-left text-sm text-slate-700 hover:bg-slate-100 dark:text-slate-200 dark:hover:bg-slate-900"
              :class="{ 'bg-slate-900 text-white hover:bg-slate-900 dark:bg-white dark:text-slate-950 dark:hover:bg-white': !hasImportanceSelection && priorityFilter === opt.value }"
              @click="selectPriorityOption(opt.value)"
            >
              {{ opt.label }}
            </button>
            <div class="my-2 border-t border-slate-200 dark:border-slate-800" />
            <div class="mb-1 px-2 text-xs font-semibold uppercase tracking-wide text-slate-400 dark:text-slate-500">
              重要度
            </div>
            <label
              v-for="n in [5, 4, 3, 2, 1]"
              :key="n"
              class="flex cursor-pointer items-center rounded-md px-2 py-2 text-sm text-slate-700 hover:bg-slate-100 dark:text-slate-200 dark:hover:bg-slate-900"
            >
              <input
                v-model="importanceChecks[n]"
                type="checkbox"
                class="mr-2"
                @change="onImportanceCheckboxChange"
              >
              重要度 = {{ n }}
            </label>
          </div>
        </div>

        <div class="relative">
          <div class="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-3">
            <i class="i-ph-magnifying-glass-bold h-4 w-4 text-slate-500 dark:text-slate-400" />
          </div>
          <input
            v-model="keyword"
            type="search"
            class="h-11 w-full rounded-lg border border-slate-200 bg-slate-50 px-3 pl-10 pr-10 text-sm text-slate-900 outline-none transition placeholder:text-slate-400 focus:border-slate-400 focus:bg-white focus:ring-2 focus:ring-slate-100 dark:border-slate-800 dark:bg-slate-900 dark:text-white dark:placeholder:text-slate-500 dark:focus:border-slate-600 dark:focus:ring-slate-800"
            placeholder="搜索单词、词义、例句或拓展"
            @keydown.stop
          >
          <button
            v-if="keyword"
            type="button"
            class="absolute inset-y-0 right-0 flex items-center pr-3 text-slate-400 hover:text-slate-700 dark:text-slate-500 dark:hover:text-slate-200"
            title="清空搜索"
            @click="keyword = ''"
          >
            <i class="i-ph-x-bold h-4 w-4" />
          </button>
        </div>

        <div class="flex flex-wrap gap-2">
          <label class="inline-flex h-11 cursor-pointer items-center gap-2 rounded-lg border border-slate-200 bg-white px-3 text-sm font-medium text-slate-700 shadow-sm dark:border-slate-800 dark:bg-slate-900 dark:text-slate-200">
            <input v-model="isTrainingModel" type="checkbox" class="peer sr-only">
            <span class="relative h-5 w-9 rounded-full bg-slate-200 transition after:absolute after:left-0.5 after:top-0.5 after:h-4 after:w-4 after:rounded-full after:bg-white after:shadow after:transition peer-checked:bg-slate-950 peer-checked:after:translate-x-4 dark:bg-slate-700 dark:peer-checked:bg-white dark:peer-checked:after:bg-slate-950" />
            练习
          </label>
          <label v-if="isTrainingModel" class="inline-flex h-11 cursor-pointer items-center gap-2 rounded-lg border border-slate-200 bg-white px-3 text-sm font-medium text-slate-700 shadow-sm dark:border-slate-800 dark:bg-slate-900 dark:text-slate-200">
            <input v-model="isShowMeaning" type="checkbox" class="peer sr-only">
            <span class="relative h-5 w-9 rounded-full bg-slate-200 transition after:absolute after:left-0.5 after:top-0.5 after:h-4 after:w-4 after:rounded-full after:bg-white after:shadow after:transition peer-checked:bg-slate-950 peer-checked:after:translate-x-4 dark:bg-slate-700 dark:peer-checked:bg-white dark:peer-checked:after:bg-slate-950" />
            释义
          </label>
          <label v-if="isTrainingModel" class="inline-flex h-11 cursor-pointer items-center gap-2 rounded-lg border border-slate-200 bg-white px-3 text-sm font-medium text-slate-700 shadow-sm dark:border-slate-800 dark:bg-slate-900 dark:text-slate-200">
            <input v-model="isShowSource" type="checkbox" class="peer sr-only">
            <span class="relative h-5 w-9 rounded-full bg-slate-200 transition after:absolute after:left-0.5 after:top-0.5 after:h-4 after:w-4 after:rounded-full after:bg-white after:shadow after:transition peer-checked:bg-slate-950 peer-checked:after:translate-x-4 dark:bg-slate-700 dark:peer-checked:bg-white dark:peer-checked:after:bg-slate-950" />
            原词
          </label>
          <label v-if="isTrainingModel" class="inline-flex h-11 cursor-pointer items-center gap-2 rounded-lg border border-slate-200 bg-white px-3 text-sm font-medium text-slate-700 shadow-sm dark:border-slate-800 dark:bg-slate-900 dark:text-slate-200">
            <input v-model="isAutoPlayWordAudio" type="checkbox" class="peer sr-only">
            <span class="relative h-5 w-9 rounded-full bg-slate-200 transition after:absolute after:left-0.5 after:top-0.5 after:h-4 after:w-4 after:rounded-full after:bg-white after:shadow after:transition peer-checked:bg-slate-950 peer-checked:after:translate-x-4 dark:bg-slate-700 dark:peer-checked:bg-white dark:peer-checked:after:bg-slate-950" />
            自动播放
          </label>
        </div>
      </div>
    </section>

    <section class="overflow-hidden rounded-xl border border-slate-200 bg-white shadow-sm dark:border-slate-800 dark:bg-slate-950">
      <div class="flex flex-col gap-4 border-b border-slate-200 bg-slate-50 p-4 dark:border-slate-800 dark:bg-slate-900/60 lg:flex-row lg:items-center lg:justify-between">
        <div class="min-w-0">
          <div class="flex flex-wrap items-center gap-2">
            <h2 class="text-lg font-bold text-slate-950 dark:text-white">
              {{ selectedCategoryLabel }}
            </h2>
            <span class="rounded-md bg-white px-2 py-1 text-xs font-semibold text-slate-500 ring-1 ring-slate-200 dark:bg-slate-950 dark:text-slate-400 dark:ring-slate-800">
              {{ selectedGroupCount }} 组
            </span>
            <span class="rounded-md bg-white px-2 py-1 text-xs font-semibold text-slate-500 ring-1 ring-slate-200 dark:bg-slate-950 dark:text-slate-400 dark:ring-slate-800">
              {{ selectedWordCount }} 个词
            </span>
            <span v-if="isFilterActive" class="rounded-md bg-slate-950 px-2 py-1 text-xs font-semibold text-white dark:bg-white dark:text-slate-950">
              匹配 {{ filteredWordCount }} 个
            </span>
          </div>
        </div>
        <audio v-if="!isAllCategories" controls class="chapter w-full max-w-80">
          <source :src="`vocabulary/audio/${refVocabulary[category].audio}`" type="audio/mpeg">
        </audio>
        <div v-else class="rounded-lg border border-slate-200 bg-white px-3 py-2 text-sm font-medium text-slate-500 dark:border-slate-800 dark:bg-slate-950 dark:text-slate-400">
          全库模式下可播放单词音频
        </div>
      </div>

      <div class="overflow-x-auto">
        <table class="min-w-full divide-y divide-slate-200 dark:divide-slate-800">
          <thead class="bg-white dark:bg-slate-950">
            <tr>
              <th class="w-16 px-4 py-3 text-left text-xs font-bold uppercase tracking-wide text-slate-500 dark:text-slate-400">
                #
              </th>
              <th class="px-4 py-3 text-left text-xs font-bold uppercase tracking-wide text-slate-500 dark:text-slate-400">
                标签
              </th>
              <th class="w-42 px-4 py-3 text-left text-xs font-bold uppercase tracking-wide text-slate-500 dark:text-slate-400">
                操作
              </th>
              <th class="px-4 py-3 text-left text-xs font-bold uppercase tracking-wide text-slate-500 dark:text-slate-400">
                词
              </th>
              <th class="px-4 py-3 text-left text-xs font-bold uppercase tracking-wide text-slate-500 dark:text-slate-400">
                词性
              </th>
              <th class="px-4 py-3 text-left text-xs font-bold uppercase tracking-wide text-slate-500 dark:text-slate-400">
                词义
              </th>
              <th class="px-4 py-3 text-left text-xs font-bold uppercase tracking-wide text-slate-500 dark:text-slate-400">
                例句
              </th>
              <th class="px-4 py-3 text-left text-xs font-bold uppercase tracking-wide text-slate-500 dark:text-slate-400">
                拓展
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 bg-white dark:divide-slate-800 dark:bg-slate-950">
            <template v-for="(wordGroup, i) of filteredWordGroups" :key="`${category}-${i}`">
              <tr
                v-for="item of wordGroup"
                v-show="(isTrainingModel && (isOnlyShowErrors ? item.spellError : true)) || !isTrainingModel"
                :id="`tr_${item.id}`"
                :key="item.id"
                :class="[`group-color-${i % 15}`]"
                class="text-sm text-slate-800 transition hover:bg-slate-50 dark:text-slate-100 dark:hover:bg-slate-900/70"
              >
                <td class="px-4 py-4 font-mono text-xs text-slate-500 dark:text-slate-400">
                  {{ isFilterActive ? filteredIndexMap.get(item.id) : item.id }}
                </td>
                <td v-if="item.frequency" class="whitespace-nowrap px-4 py-4">
                  <div class="flex flex-col items-start gap-1.5">
                    <span
                      class="inline-flex items-center rounded-md px-2 py-0.5 text-xs font-semibold"
                      :class="FREQUENCY_CLASSES[item.frequency]"
                      :title="`出现频率：${FREQUENCY_LABELS[item.frequency]}`"
                    >{{ FREQUENCY_LABELS[item.frequency] }}</span>
                    <span
                      class="inline-flex items-center rounded-md bg-amber-100 px-2 py-0.5 text-xs font-semibold text-amber-800 dark:bg-amber-950 dark:text-amber-200"
                      :title="`重要度：${item.importance}/5${item.reason ? ` · ${item.reason}` : ''}`"
                    >★{{ item.importance }}</span>
                    <span
                      class="inline-flex items-center rounded-md px-2 py-0.5 text-xs font-semibold"
                      :class="MASTERY_CLASSES[item.mastery]"
                      :title="item.mastery === 'productive' ? '需要能够主动写出/说出' : '只需要能够识别、理解'"
                    >{{ MASTERY_LABELS[item.mastery] }}</span>
                  </div>
                </td>
                <td v-else class="px-4 py-4" />
                <td class="whitespace-nowrap px-4 py-4">
                  <button
                    type="button"
                    class="inline-grid h-9 w-9 place-items-center rounded-lg border border-slate-200 bg-white text-slate-600 transition hover:border-slate-300 hover:text-slate-950 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-300 dark:hover:text-white"
                    title="播放单词"
                    @click="play(getWordAudioPath(item))"
                  >
                    <i class="i-ph-speaker-simple-high-bold" />
                  </button>

                  <template v-if="isTrainingModel">
                    <button
                      type="button"
                      class="ml-2 inline-grid h-9 w-9 place-items-center rounded-lg border border-slate-200 bg-white text-slate-600 transition hover:border-slate-300 hover:text-slate-950 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-300 dark:hover:text-white"
                      title="显示原词"
                      @click="item.showSource = !item.showSource"
                    >
                      <i :class="item.showSource ? 'i-ph-eye-slash-bold' : 'i-ph-eye-bold'" />
                    </button>
                    <input
                      :id="item.id"
                      autocomplete="off"
                      :class="getInputStyleClass(item)"
                      type="text"
                      @focusout="onInputFocusOut($event, item)"
                      @focusin="onInputFocusIn($event, getWordAudioPath(item))"
                      @keydown="onInputKeydown"
                    >
                  </template>
                </td>
                <td class="group relative whitespace-nowrap px-4 py-4 font-semibold text-slate-950 dark:text-white">
                  <div v-if="!isTrainingModel || item.showSource || (isTrainingModel && isOnlyShowErrors && item.spellError) || isShowSource">
                    <p v-for="w in item.word" :key="w">
                      <a
                        class="decoration-slate-300 underline-offset-4 hover:underline dark:decoration-slate-600"
                        :title="`在剑桥词典中查询 ${w}`"
                        target="_blank"
                        :href="`https://dictionary.cambridge.org/dictionary/english-chinese-simplified/${w}`"
                      >{{ w }}</a>
                    </p>

                    <button
                      type="button"
                      class="absolute right-1 top-1 hidden h-8 w-8 place-items-center rounded-md text-slate-400 hover:bg-slate-100 hover:text-slate-900 group-hover:grid dark:hover:bg-slate-800 dark:hover:text-white"
                      title="复制词条"
                      @click="copyText(item)"
                    >
                      <i class="i-ph-copy" />
                    </button>
                  </div>
                </td>
                <td class="whitespace-nowrap px-4 py-4 font-serif italic text-slate-500 dark:text-slate-400">
                  {{ item.pos }}
                </td>
                <td class="min-w-56 px-4 py-4 text-slate-700 dark:text-slate-200">
                  {{ isShowMeaning ? item.meaning : '' }}
                </td>
                <td class="min-w-90 px-4 py-4 text-slate-600 dark:text-slate-300">
                  {{ isTrainingModel ? '' : item.example }}
                </td>
                <td class="min-w-64 px-4 py-4">
                  <div
                    v-if="!isTrainingModel"
                    class="min-h-9 rounded-lg border border-transparent px-3 py-2 text-slate-600 outline-none transition hover:border-slate-200 hover:bg-slate-50 focus:border-slate-300 focus:bg-white focus:ring-2 focus:ring-slate-100 dark:text-slate-300 dark:hover:border-slate-800 dark:hover:bg-slate-900 dark:focus:border-slate-700 dark:focus:bg-slate-950 dark:focus:ring-slate-800"
                    contenteditable="true"
                    @blur="onExtraBlur($event, item)"
                  >
                    {{ item.extra }}
                  </div>
                </td>
              </tr>
            </template>
            <tr v-if="isFilterActive && filteredWordCount === 0">
              <td
                colspan="8"
                class="px-4 py-12 text-center text-sm font-medium text-slate-500 dark:text-slate-300"
              >
                没有找到匹配的单词
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <section v-if="isTrainingModel" class="flex flex-col gap-3 rounded-xl border border-slate-200 bg-white p-4 shadow-sm dark:border-slate-800 dark:bg-slate-950 sm:flex-row sm:items-center sm:justify-between">
      <p class="text-sm font-medium text-slate-600 dark:text-slate-300">
        {{ trainingStats || '完成后会在这里显示练习统计' }}
      </p>
      <div class="flex flex-wrap gap-2">
        <button
          type="button"
          class="inline-flex items-center gap-2 rounded-lg bg-slate-950 px-4 py-2.5 text-sm font-semibold text-white transition hover:bg-slate-800 dark:bg-white dark:text-slate-950 dark:hover:bg-slate-200"
          @click="isFinishTraining = true"
        >
          <i class="i-carbon-checkmark" />
          完成练习
        </button>
        <button
          type="button"
          class="inline-flex items-center gap-2 rounded-lg border border-slate-200 bg-white px-4 py-2.5 text-sm font-semibold text-slate-700 transition hover:border-slate-300 hover:text-slate-950 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-200 dark:hover:text-white"
          @click="isOnlyShowErrors = !isOnlyShowErrors"
        >
          <i class="i-carbon-filter" />
          {{ isOnlyShowErrors ? '展示所有' : '仅展示错词' }}
        </button>
        <button
          type="button"
          class="inline-flex items-center gap-2 rounded-lg border border-slate-200 bg-white px-4 py-2.5 text-sm font-semibold text-slate-700 transition hover:border-slate-300 hover:text-slate-950 dark:border-slate-800 dark:bg-slate-900 dark:text-slate-200 dark:hover:text-white"
          @click="copyAllError"
        >
          <i class="i-ph-copy" />
          拷贝错词
        </button>
      </div>
    </section>
  </div>
</template>
