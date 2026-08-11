<!-- eslint-disable eslint-comments/no-unlimited-disable -->
<script setup generic="T extends any, O extends any">
import vocabulary from './vocabulary'

const CHAPTER_KEY = 'vocabulary_chapter'
const EXTRA_OVERRIDES_KEY = 'vocabulary_extra_overrides'

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
const chapters = Object.keys(vocabulary)
const category = ref(localStorage.getItem(CHAPTER_KEY) || chapters[0])

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
const currentChapter = computed(() => refVocabulary[category.value])
const isPriorityFilterActive = computed(() => hasImportanceSelection.value || priorityFilter.value !== 'all')
const isFilterActive = computed(() => isPriorityFilterActive.value || !!searchKeyword.value)

const extraOverrides = loadExtraOverrides()
for (const cat of Object.values(refVocabulary)) {
  for (const group of cat.words) {
    for (const item of group) {
      if (extraOverrides[item.id] !== undefined)
        item.extra = extraOverrides[item.id]
    }
  }
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
  const chapter = currentChapter.value
  if (!chapter)
    return []

  return chapter.words
    .map(group => group.filter(item => matchesWordFilters(item)))
    .filter(group => group.length > 0)
})

const filteredWordCount = computed(() => {
  return filteredWordGroups.value.reduce((total, group) => total + group.length, 0)
})

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
    const cur = refVocabulary[category.value]
    // 遍历所有单词的属性
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
    error: 'ml-4 bg-red-50 border border-red-500 text-red-900 placeholder-red-700 text-sm rounded-lg focus:ring-red-500 dark:bg-gray-700 focus:border-red-500 inline-block p-2.5 dark:text-red-500 dark:placeholder-red-500 dark:border-red-500',
    normal: 'ml-4 inline-block border border-gray-300 rounded-lg bg-gray-50 p-2.5 text-sm text-gray-900 dark:border-gray-600 focus:border-blue-500 dark:bg-gray-700 dark:text-white focus:ring-blue-500 dark:focus:border-blue-500 dark:focus:ring-blue-500 dark:placeholder-gray-400',
    success: 'ml-4 bg-green-50 border border-green-500 text-green-900 dark:text-green-400 placeholder-green-700 dark:placeholder-green-500 text-sm rounded-lg focus:ring-green-500 focus:border-green-500 inline-block p-2.5 dark:bg-gray-700 dark:border-green-500',
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
  const words = refVocabulary[category.value].words
  const errorWords = []
  for (const group of words) {
    for (const item of group) {
      if (item.spellError)
        errorWords.push(`${item.word} ${item.pos} ${item.meaning}`)
    }
  }
  navigator.clipboard.writeText(errorWords.join('\n\n'))
}
</script>

<template>
  <div class="px-4 pt-6 2xl:px-0">
    <div class="border border-gray-200 rounded-lg bg-white p-4 shadow-sm dark:border-gray-700 dark:bg-gray-800 sm:p-6">
      <!-- Card header -->
      <div class="items-center justify-between lg:flex">
        <div class="mb-4 lg:mb-0">
          <h3 class="mb-2 text-xl font-bold text-gray-900 dark:text-white">
            雅思词汇真经
          </h3>
          <span class="text-base font-normal text-gray-500 dark:text-gray-400">涵盖雅思必备核心词，逻辑词群记忆法</span>
        </div>
        <div class="items-center sm:flex">
          <div class="flex flex-wrap items-center gap-2">
            <select
              v-model="category"
              class="block min-w-48 flex-1 border border-gray-300 rounded-lg bg-gray-50 p-2.5 text-sm text-gray-900 dark:border-gray-600 focus:border-blue-500 dark:bg-gray-700 dark:text-white focus:ring-blue-500 dark:focus:border-blue-500 dark:focus:ring-blue-500 dark:placeholder-gray-400"
            >
              <!-- <option value="">
                全部章节
              </option> -->
              <option v-for="(_, k) in refVocabulary" :key="k" :value="k">
                {{ k }}
              </option>
            </select>
            <div ref="filterDropdownRef" class="relative">
              <button
                type="button"
                class="w-56 flex items-center justify-between border border-gray-300 rounded-lg bg-gray-50 p-2.5 text-sm text-gray-900 dark:border-gray-600 dark:bg-gray-700 dark:text-white"
                aria-haspopup="true" :aria-expanded="isFilterOpen"
                @click="isFilterOpen = !isFilterOpen"
              >
                <span class="truncate">{{ filterSummaryLabel }}</span>
                <i class="i-ph-caret-down-bold ml-2 flex-shrink-0" />
              </button>
              <div
                v-show="isFilterOpen"
                class="absolute z-10 mt-1 w-64 border border-gray-200 rounded-lg bg-white p-2 shadow-lg dark:border-gray-700 dark:bg-gray-800"
              >
                <div class="mb-1 px-2 text-xs font-semibold text-gray-400 dark:text-gray-500">
                  按条件筛选
                </div>
                <button
                  v-for="opt in PRIORITY_OPTIONS" :key="opt.value"
                  type="button"
                  class="block w-full rounded px-2 py-1.5 text-left text-sm hover:bg-gray-100 dark:hover:bg-gray-700"
                  :class="{ 'bg-blue-50 text-blue-700 dark:bg-blue-900 dark:text-blue-300': !hasImportanceSelection && priorityFilter === opt.value }"
                  @click="selectPriorityOption(opt.value)"
                >
                  {{ opt.label }}
                </button>
                <div class="my-2 border-t border-gray-200 dark:border-gray-700" />
                <div class="mb-1 px-2 text-xs font-semibold text-gray-400 dark:text-gray-500">
                  按重要度多选
                </div>
                <label
                  v-for="n in [5, 4, 3, 2, 1]" :key="n"
                  class="flex cursor-pointer items-center rounded px-2 py-1.5 text-sm hover:bg-gray-100 dark:hover:bg-gray-700"
                >
                  <input
                    v-model="importanceChecks[n]" type="checkbox" class="mr-2"
                    @change="onImportanceCheckboxChange"
                  >
                  重要度 = {{ n }}
                </label>
              </div>
            </div>
            <div class="relative min-w-64 flex-1 sm:flex-none">
              <div class="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-3">
                <i class="i-ph-magnifying-glass-bold h-4 w-4 text-gray-500 dark:text-gray-400" />
              </div>
              <input
                v-model="keyword"
                type="search"
                class="block w-full border border-gray-300 rounded-lg bg-gray-50 p-2.5 pl-10 pr-10 text-sm text-gray-900 dark:border-gray-600 focus:border-blue-500 dark:bg-gray-700 dark:text-white focus:ring-blue-500 dark:focus:border-blue-500 dark:focus:ring-blue-500 dark:placeholder-gray-400"
                placeholder="搜索单词/词义/例句"
                @keydown.stop
              >
              <button
                v-if="keyword"
                type="button"
                class="absolute inset-y-0 right-0 flex items-center pr-3 text-gray-400 dark:text-gray-500 hover:text-gray-700 dark:hover:text-gray-200"
                title="清空搜索"
                @click="keyword = ''"
              >
                <i class="i-ph-x-bold h-4 w-4" />
              </button>
            </div>
            <label class="ml-2 inline-flex cursor-pointer items-center">
              <input v-model="isTrainingModel" type="checkbox" class="peer sr-only">
              <div
                class="peer relative h-6 w-11 rounded-full bg-gray-200 after:absolute after:start-[2px] after:top-[2px] after:h-5 after:w-5 after:border after:border-gray-300 dark:border-gray-600 after:rounded-full after:bg-white dark:bg-gray-700 peer-checked:bg-blue-600 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 after:transition-all after:content-[''] peer-checked:after:translate-x-full peer-checked:after:border-white dark:peer-focus:ring-blue-800 rtl:peer-checked:after:-translate-x-full"
              />
              <span class="ms-3 text-sm font-medium text-gray-900 dark:text-gray-300">练习模式</span>
            </label>
            <label v-if="isTrainingModel" class="ml-2 inline-flex cursor-pointer items-center">
              <input v-model="isShowMeaning" type="checkbox" class="peer sr-only">
              <div
                class="peer relative h-6 w-11 rounded-full bg-gray-200 after:absolute after:start-[2px] after:top-[2px] after:h-5 after:w-5 after:border after:border-gray-300 dark:border-gray-600 after:rounded-full after:bg-white dark:bg-gray-700 peer-checked:bg-blue-600 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 after:transition-all after:content-[''] peer-checked:after:translate-x-full peer-checked:after:border-white dark:peer-focus:ring-blue-800 rtl:peer-checked:after:-translate-x-full"
              />
              <span class="ms-3 text-sm font-medium text-gray-900 dark:text-gray-300">释义</span>
            </label>
            <label v-if="isTrainingModel" class="ml-2 inline-flex cursor-pointer items-center">
              <input v-model="isShowSource" type="checkbox" class="peer sr-only">
              <div
                class="peer relative h-6 w-11 rounded-full bg-gray-200 after:absolute after:start-[2px] after:top-[2px] after:h-5 after:w-5 after:border after:border-gray-300 dark:border-gray-600 after:rounded-full after:bg-white dark:bg-gray-700 peer-checked:bg-blue-600 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 after:transition-all after:content-[''] peer-checked:after:translate-x-full peer-checked:after:border-white dark:peer-focus:ring-blue-800 rtl:peer-checked:after:-translate-x-full"
              />
              <span class="ms-3 text-sm font-medium text-gray-900 dark:text-gray-300">原词</span>
            </label>
            <label v-if="isTrainingModel" class="ml-2 inline-flex cursor-pointer items-center">
              <input v-model="isAutoPlayWordAudio" type="checkbox" class="peer sr-only">
              <div
                class="peer relative h-6 w-11 rounded-full bg-gray-200 after:absolute after:start-[2px] after:top-[2px] after:h-5 after:w-5 after:border after:border-gray-300 dark:border-gray-600 after:rounded-full after:bg-white dark:bg-gray-700 peer-checked:bg-blue-600 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 after:transition-all after:content-[''] peer-checked:after:translate-x-full peer-checked:after:border-white dark:peer-focus:ring-blue-800 rtl:peer-checked:after:-translate-x-full"
              />
              <span class="ms-3 text-sm font-medium text-gray-900 dark:text-gray-300">自动播放</span>
            </label>
          </div>
        </div>
      </div>
      <!-- Table -->
      <div class="mt-6 flex flex-col">
        <div class="overflow-x-auto rounded-lg">
          <div class="inline-block min-w-full align-middle">
            <div class="overflow-hidden shadow sm:rounded-lg">
              <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-600">
                <thead class="bg-gray-50 dark:bg-gray-700">
                  <tr>
                    <th class="p-4 text-left text-xs font-medium tracking-wider text-gray-500 dark:text-white">
                      #
                    </th>
                    <th class="p-4 text-left text-xs font-medium tracking-wider text-gray-500 dark:text-white">
                      标签
                    </th>
                    <th class="p-4 text-xs font-medium tracking-wider text-gray-500 dark:text-white">
                      <br>
                    </th>
                    <th class="p-4 text-left text-xs font-medium tracking-wider text-gray-500 dark:text-white">
                      词
                    </th>
                    <th class="w-0 text-left text-xs font-medium text-gray-500 dark:text-white">
                      词性
                    </th>
                    <th class="p-4 text-left text-xs font-medium tracking-wider text-gray-500 dark:text-white">
                      词义
                    </th>
                    <th class="p-4 text-left text-xs font-medium tracking-wider text-gray-500 dark:text-white">
                      例句
                    </th>
                    <th class="p-4 text-left text-xs font-medium tracking-wider text-gray-500 dark:text-white">
                      拓展
                    </th>
                  </tr>
                </thead>
                <tbody class="bg-white dark:bg-gray-800">
                  <tr class="bg-hex-f3f3f3">
                    <td
                      colspan="8"
                      class="px-4 py-6 text-sm font-normal text-gray-900 dark:bg-gray-500 dark:text-white"
                    >
                      <div class="flex flex-row">
                        <div class="flex flex-1 items-center">
                          <span class="text-lg">{{ category }}</span>
                          （ {{ refVocabulary[category].groupCount }} 组 {{ refVocabulary[category].wordCount }} 个词 ）
                          <span v-if="isFilterActive" class="ml-2 text-gray-500 dark:text-gray-200">
                            已筛选/匹配 {{ filteredWordCount }} 个词
                          </span>
                        </div>
                        <div class="justify-items-end">
                          <audio controls class="chapter">
                            <source :src="`vocabulary/audio/${refVocabulary[category].audio}`" type="audio/mpeg">
                          </audio>
                        </div>
                      </div>
                    </td>
                  </tr>
                  <template v-for="(wordGroup, i) of filteredWordGroups" :key="`${category}-${i}`">
                    <tr
                      v-for="item of wordGroup"
                      v-show="(isTrainingModel && (isOnlyShowErrors ? item.spellError : true)) || !isTrainingModel" :id="`tr_${item.id}`"
                      :key="item.id"
                      :class="{ 'bg-gray-50 dark:bg-gray-700': item.id % 2 === 0, [`group-color-${i % 15}`]: true }" class="text-sm text-gray-900 dark:text-white"
                    >
                      <td class="p-4">
                        {{ isFilterActive ? filteredIndexMap.get(item.id) : item.id }}
                      </td>
                      <td v-if="item.frequency" class="whitespace-nowrap p-4">
                        <div class="flex flex-col items-start gap-1">
                          <span
                            class="inline-flex items-center rounded-full px-2 py-0.5 text-xs font-medium"
                            :class="FREQUENCY_CLASSES[item.frequency]"
                            :title="`出现频率：${FREQUENCY_LABELS[item.frequency]}`"
                          >{{ FREQUENCY_LABELS[item.frequency] }}</span>
                          <span
                            class="inline-flex items-center rounded-full bg-orange-100 px-2 py-0.5 text-xs font-medium text-orange-800 dark:bg-orange-900 dark:text-orange-300"
                            :title="`重要度：${item.importance}/5${item.reason ? ` · ${item.reason}` : ''}`"
                          >★{{ item.importance }}</span>
                          <span
                            class="inline-flex items-center rounded-full px-2 py-0.5 text-xs font-medium"
                            :class="MASTERY_CLASSES[item.mastery]"
                            :title="item.mastery === 'productive' ? '需要能够主动写出/说出' : '只需要能够识别、理解'"
                          >{{ MASTERY_LABELS[item.mastery] }}</span>
                        </div>
                      </td>
                      <td v-else class="p-4" />
                      <td>
                        <i
                          class="i-ph-speaker-simple-high-bold inline-block cursor-pointer"
                          @click="play(`vocabulary/audio/${category}/${item.word[0]}.mp3`)"
                        />

                        <template v-if="isTrainingModel">
                          <i
                            :class="`${item.showSource ? 'i-ph-eye-slash-bold' : 'i-ph-eye-bold'} inline-block cursor-pointer ml-4`"
                            title="显示原词" @click="item.showSource = !item.showSource"
                          />
                          <input
                            :id="item.id" autocomplete="off" :class="getInputStyleClass(item)"
                            type="text"
                            @focusout="onInputFocusOut($event, item)"
                            @focusin="onInputFocusIn($event, `vocabulary/audio/${category}/${item.word[0]}.mp3`)"
                            @keydown="onInputKeydown"
                          >
                        </template>
                      </td>
                      <td class="group relative whitespace-nowrap p-4">
                        <div v-if="!isTrainingModel || item.showSource || (isTrainingModel && isOnlyShowErrors && item.spellError) || isShowSource">
                          <p v-for="w in item.word" :key="w">
                            <a
                              class="hover:underline" :title="`在剑桥词典中查询 ${w}`" target="_blank"
                              :href="`https://dictionary.cambridge.org/dictionary/english-chinese-simplified/${w}`"
                            >{{ w }}</a>
                          </p>

                          <div
                            class="absolute right-0 top-0 hidden h-100% items-center group-hover:flex"
                            @click="copyText(item)"
                          >
                            <i class="i-ph-copy block cursor-pointer px-4" />
                          </div>
                        </div>
                      </td>
                      <td style="font-style: italic; font-family: times;">
                        {{ item.pos }}
                      </td>
                      <td class="p-4">
                        {{ isShowMeaning ? item.meaning : '' }}
                      </td>
                      <td class="p-4">
                        {{ isTrainingModel ? '' : item.example }}
                      </td>
                      <td class="p-4">
                        <div
                          v-if="!isTrainingModel"
                          class="min-w-20 rounded p-1 outline-none focus:bg-white hover:bg-gray-50 focus:ring-1 focus:ring-blue-500 dark:focus:bg-gray-700 dark:hover:bg-gray-700"
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
                      class="px-4 py-8 text-center text-sm font-normal text-gray-500 dark:bg-gray-800 dark:text-gray-300"
                    >
                      没有找到匹配的单词
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
      <!-- Card Footer -->
      <div class="flex items-center justify-between pt-3 sm:pt-6">
        <div>
          <p v-if="isTrainingModel">
            {{ trainingStats }}
          </p>
        </div>
        <div v-if="isTrainingModel" class="flex-shrink-0">
          <button
            type="button"
            class="rounded-lg bg-blue-700 px-5 py-2.5 text-sm font-medium text-white dark:bg-blue-600 hover:bg-blue-800 focus:outline-none focus:ring-4 focus:ring-blue-300 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
            @click="isFinishTraining = true"
          >
            完成练习
          </button>
          <button
            type="button"
            class="ml-2 rounded-lg bg-blue-700 px-5 py-2.5 text-sm font-medium text-white dark:bg-blue-600 hover:bg-blue-800 focus:outline-none focus:ring-4 focus:ring-blue-300 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
            @click="isOnlyShowErrors = !isOnlyShowErrors"
          >
            {{ isOnlyShowErrors ? '展示所有' : '仅展示错词' }}
          </button>
          <button
            type="button"
            class="ml-2 rounded-lg bg-blue-700 px-5 py-2.5 text-sm font-medium text-white dark:bg-blue-600 hover:bg-blue-800 focus:outline-none focus:ring-4 focus:ring-blue-300 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
            @click="copyAllError"
          >
            拷贝错词
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
