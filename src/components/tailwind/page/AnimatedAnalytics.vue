<template>
  <div class="h-screen">
    <analytics-layout>
      <!-- 固定在顶部的导航 -->
      <div class="fixed top-0 left-0 right-0 z-50">
        <div class="bg-white/5 backdrop-blur-md border-b border-white/10 dark:bg-black/5 dark:border-gray-800/50">
          <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between items-center h-16">
              <!-- 添加返回按钮 -->
              <div class="flex items-center">
                <button
                  @click="goToHome"
                  class="mr-3 p-1 rounded-full hover:bg-white/20 dark:hover:bg-black/20 transition-colors"
                  title="返回首页"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-[#fb7299] dark:text-[#fc9b7a]" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" />
                  </svg>
                </button>
                <h1 class="text-2xl font-bold bg-gradient-to-r from-[#fb7299] to-[#fc9b7a] bg-clip-text text-transparent">
                  {{ selectedYear }}年度回顾
                </h1>
              </div>
              <div class="flex items-center space-x-4">
                <select
                  v-model="selectedYear"
                  class="w-24 bg-white/10 backdrop-blur text-gray-800 dark:text-white border border-white/20 dark:border-gray-700 rounded-lg px-3 py-1 focus:ring-2 focus:ring-[#fb7299] focus:border-transparent transition-colors duration-200"
                  :disabled="loading"
                >
                  <option v-for="year in availableYears" :key="year" :value="year">
                    {{ year }}年
                  </option>
                </select>

                <!-- 强制刷新按钮 -->
                <button
                  @click="handleForceRefresh"
                  :disabled="loading"
                  class="inline-flex items-center text-gray-600 dark:text-gray-300 hover:text-[#fb7299] dark:hover:text-[#fb7299] disabled:opacity-50 disabled:cursor-not-allowed transition-colors duration-200"
                >
                  <svg
                    class="w-5 h-5"
                    :class="{'animate-spin': loading}"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <circle
                      v-if="loading"
                      class="opacity-25"
                      cx="12"
                      cy="12"
                      r="10"
                      stroke="currentColor"
                      stroke-width="4"
                    ></circle>
                    <path
                      v-if="loading"
                      class="opacity-75"
                      fill="currentColor"
                      d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                    ></path>
                    <path
                      v-if="!loading"
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
                    ></path>
                  </svg>
                  <span class="ml-2">{{ loading ? '加载中' : '强制刷新' }}</span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 主要内容区域 -->
      <div class="relative h-full pt-16">
        <!-- 页面容器 -->
        <div class="h-full">
          <!-- 加载状态 -->
          <div v-if="loading"
            class="fixed inset-0 flex items-center justify-center z-50 bg-white/80 dark:bg-gray-900/80 backdrop-blur-sm"
            style="min-height: 100vh"
          >
            <div class="text-center">
              <svg
                class="w-12 h-12 mx-auto mb-4 animate-spin text-[#fb7299]"
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
              >
                <circle
                  class="opacity-25"
                  cx="12"
                  cy="12"
                  r="10"
                  stroke="currentColor"
                  stroke-width="4"
                ></circle>
                <path
                  class="opacity-75"
                  fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                ></path>
              </svg>
              <div class="space-y-2">
                <p class="text-lg font-medium text-gray-800 dark:text-gray-200">正在分析{{ selectedYear }}年的观看数据</p>
                <p class="text-sm text-gray-600 dark:text-gray-400">
                  {{ loading && viewingData === null ? '首次加载数据可能需要30秒到1分钟，具体加载时间取决于数据量' : '正在从缓存加载数据，预计3-5秒' }}
                </p>
              </div>
            </div>
          </div>

          <!-- 内容页面 -->
          <Transition mode="out-in" name="fade">
            <!-- 开场页 -->
            <HeroPage v-if="currentPage === 0" key="hero" :year="selectedYear" />

            <!-- 数据概览页 -->
            <OverviewPage v-else-if="currentPage === 1" key="overview" :viewing-data="viewingData" />

            <!-- 时间分析页 -->
            <TimeAnalysisPage v-else-if="currentPage === 2" key="time-analysis" :viewing-data="viewingData" :selected-year="selectedYear" />

            <!-- 时间分布分析页 -->
            <TimeDistributionPage v-else-if="currentPage === 3" key="time-distribution" :viewing-data="viewingData" />

            <!-- 月度趋势页 -->
            <MonthlyPage v-else-if="currentPage === 4" key="monthly" :viewing-data="viewingData" />

            <!-- 连续观看页 -->
            <StreakPage v-else-if="currentPage === 5" key="streak" :viewing-data="viewingData" :selected-year="selectedYear" />

            <!-- 最爱重温页 -->
            <RewatchPage v-else-if="currentPage === 6" key="rewatch" :viewing-data="viewingData" />

            <!-- 视频完成率分析页 -->
            <OverallCompletionPage v-else-if="currentPage === 7" key="overall-completion" :viewing-data="viewingData" />

            <!-- UP主完成率分析页 -->
            <AuthorCompletionPage v-else-if="currentPage === 8" key="author-completion" :viewing-data="viewingData" />

            <!-- 标签分析页 -->
            <TagsPage v-else-if="currentPage === 9" key="tags" :viewing-data="viewingData" />

            <!-- 视频时长分析页 -->
            <DurationAnalysisPage v-else-if="currentPage === 10" key="duration-analysis" :viewing-data="viewingData" />

            <!-- 标题分析页 -->
            <TitleAnalysisPage v-else-if="currentPage === 11" key="title-analysis" :title-analytics="analyticsData" />

            <!-- 标题趋势分析页 -->
            <TitleTrendAnalysisPage v-else-if="currentPage === 12" key="title-trend-analysis" :title-analytics="analyticsData" />

            <!-- 标题长度分析页 -->
            <TitleLengthAnalysisPage v-else-if="currentPage === 13" key="title-length-analysis" :title-analytics="analyticsData" />

            <!-- 标题情感分析页 -->
            <TitleSentimentAnalysisPage v-else-if="currentPage === 14" key="title-sentiment-analysis" :title-analytics="analyticsData" />

            <!-- 标题互动分析页 -->
            <TitleInteractionAnalysisPage v-else-if="currentPage === 15" key="title-interaction-analysis" :title-analytics="analyticsData" />
          </Transition>
        </div>
      </div>
    </analytics-layout>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'
import { getTitleAnalytics, getViewingAnalytics } from '../../../api/api.js'
import HeroPage from '../analytics/pages/HeroPage.vue'
import OverviewPage from '../analytics/pages/OverviewPage.vue'
import StreakPage from '../analytics/pages/StreakPage.vue'
import TimeAnalysisPage from '../analytics/pages/TimeAnalysisPage.vue'
import RewatchPage from '../analytics/pages/RewatchPage.vue'
import OverallCompletionPage from '../analytics/pages/OverallCompletionPage.vue'
import AuthorCompletionPage from '../analytics/pages/AuthorCompletionPage.vue'
import TagsPage from '../analytics/pages/TagsPage.vue'
import TimeDistributionPage from '../analytics/pages/TimeDistributionPage.vue'
import MonthlyPage from '../analytics/pages/MonthlyPage.vue'
import DurationAnalysisPage from '../analytics/pages/DurationAnalysisPage.vue'
import TitleAnalysisPage from '../analytics/pages/TitleAnalysisPage.vue'
import TitleLengthAnalysisPage from '../analytics/pages/TitleLengthAnalysisPage.vue'
import TitleSentimentAnalysisPage from '../analytics/pages/TitleSentimentAnalysisPage.vue'
import TitleTrendAnalysisPage from '../analytics/pages/TitleTrendAnalysisPage.vue'
import TitleInteractionAnalysisPage from '../analytics/pages/TitleInteractionAnalysisPage.vue'
import AnalyticsLayout from '../analytics/layout/AnalyticsLayout.vue'
import { CanvasRenderer } from 'echarts/renderers'
import { LineChart, BarChart } from 'echarts/charts'
import {
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent
} from 'echarts/components'
import { use } from 'echarts/core'
import 'echarts-wordcloud'
import { useRouter, useRoute } from 'vue-router'

// 注册必要的组件
use([
  CanvasRenderer,
  LineChart,
  BarChart,
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent
])

// 状态
const router = useRouter()
const route = useRoute()
const selectedYear = ref(new Date().getFullYear())
const availableYears = ref([])
const loading = ref(true)
const analyticsData = ref(null)
const viewingData = ref(null)
const currentPage = ref(0)
const isTransitioning = ref(false)

// 滚动和触摸相关状态
let touchStartX = 0
let touchStartY = 0
let lastWheelTime = 0
const wheelThreshold = 30 // 降低滚动阈值
const wheelCooldown = 800 // 增加冷却时间以适应新的动画持续时间

// 页面配置
const pages = [
  { name: '开场', color: '#fb7299' },
  { name: '数据概览', color: '#fc9b7a' },
  // 时间相关分析
  { name: '时间分析', color: '#fb7299' },
  { name: '时间分布', color: '#fc9b7a' },
  { name: '月度趋势', color: '#fb7299' },
  // 观看行为分析
  { name: '连续观看', color: '#fc9b7a' },
  { name: '最爱重温', color: '#fb7299' },
  { name: '视频完成率', color: '#fc9b7a' },
  // 内容分析
  { name: 'UP主完成率', color: '#fb7299' },
  { name: '标签分析', color: '#fc9b7a' },
  { name: '视频时长分析', color: '#fb7299' },
  // 标题分析
  { name: '标题分析', color: '#fc9b7a' },
  { name: '标题趋势分析', color: '#fb7299' },
  { name: '标题长度分析', color: '#fc9b7a' },
  { name: '标题情感分析', color: '#fb7299' },
  { name: '标题互动分析', color: '#fc9b7a' }
]

// 监听页面切换
watch(currentPage, (newPage) => {
  // 减少过渡动画时间
  isTransitioning.value = true
  setTimeout(() => {
    isTransitioning.value = false
  }, 300)

  // 更新 URL
  router.push({
    query: {
      ...route.query,
      page: newPage
    }
  })
})

// 修改页面切换处理
const handlePageTransition = (newPage) => {
  if (isTransitioning.value) return
  currentPage.value = newPage
}

// 修改滚轮事件处理
const handleWheel = (e) => {
  // 如果正在加载，阻止滚动
  if (loading.value) {
    e.preventDefault()
    return
  }

  const now = Date.now()
  if (isTransitioning.value || now - lastWheelTime < wheelCooldown) return

  const deltaY = Math.abs(e.deltaY)
  if (deltaY < wheelThreshold) return

  lastWheelTime = now

  if (e.deltaY > 0 && currentPage.value < pages.length - 1) {
    handlePageTransition(currentPage.value + 1)
  } else if (e.deltaY < 0 && currentPage.value > 0) {
    handlePageTransition(currentPage.value - 1)
  }
}

// 修改触摸事件处理
const handleTouchStart = (e) => {
  // 如果正在加载，阻止触摸事件
  if (loading.value || isTransitioning.value) return
  touchStartX = e.touches[0].clientX
  touchStartY = e.touches[0].clientY
}

const handleTouchMove = (e) => {
  // 如果正在加载，阻止触摸移动
  if (loading.value) {
    e.preventDefault()
    return
  }

  if (isTransitioning.value) return
  e.preventDefault() // 防止页面滚动
}

const handleTouchEnd = (e) => {
  // 如果正在加载，阻止触摸结束事件
  if (loading.value || isTransitioning.value) return

  const touchEndX = e.changedTouches[0].clientX
  const touchEndY = e.changedTouches[0].clientY
  const deltaX = touchEndX - touchStartX
  const deltaY = touchEndY - touchStartY

  if (Math.abs(deltaY) > Math.abs(deltaX)) {
    if (Math.abs(deltaY) < 50) return

    if (deltaY < 0 && currentPage.value < pages.length - 1) {
      handlePageTransition(currentPage.value + 1)
    } else if (deltaY > 0 && currentPage.value > 0) {
      handlePageTransition(currentPage.value - 1)
    }
  }
}

// 获取数据方法
const fetchAnalyticsData = async (forceRefresh = false) => {
  if (forceRefresh) {
    // 清空现有数据
    analyticsData.value = null
    viewingData.value = null
  }

  loading.value = true
  try {
    console.log('Fetching analytics data for year:', selectedYear.value)
    const [titleResponse, viewingResponse] = await Promise.all([
      getTitleAnalytics(selectedYear.value, !forceRefresh),
      getViewingAnalytics(selectedYear.value, !forceRefresh)
    ])

    if (titleResponse.data.status === 'success') {
      analyticsData.value = titleResponse.data.data
      // 更新可用年份
      if (titleResponse.data.data.available_years) {
        availableYears.value = titleResponse.data.data.available_years
        if (!availableYears.value.includes(selectedYear.value)) {
          selectedYear.value = availableYears.value[0]
        }
      }
    }

    if (viewingResponse.data.status === 'success') {
      viewingData.value = viewingResponse.data.data
    }
  } catch (error) {
    console.error('获取分析数据失败:', error)
  } finally {
    loading.value = false
  }
}

// 移除单独的年份获取方法
const refreshData = async (forceRefresh = false) => {
  loading.value = true
  try {
    await fetchAnalyticsData(forceRefresh)
  } finally {
    loading.value = false
  }
}

// 强制刷新方法
const handleForceRefresh = async () => {
  if (loading.value) return
  await refreshData(true)
}

// 监听年份变化
watch(selectedYear, async (newYear) => {
  if (newYear) {
    await refreshData()
  }
})

// 生命周期钩子
onMounted(async () => {
  // 从 URL 读取页码
  const pageFromUrl = parseInt(Array.isArray(route.query.page) ? route.query.page[0] : route.query.page || '0')
  if (!isNaN(pageFromUrl) && pageFromUrl >= 0 && pageFromUrl < pages.length) {
    currentPage.value = pageFromUrl
  }

  await refreshData()
  window.addEventListener('wheel', handleWheel, { passive: false })
  window.addEventListener('touchstart', handleTouchStart)
  window.addEventListener('touchmove', handleTouchMove, { passive: false })
  window.addEventListener('touchend', handleTouchEnd)
})

onUnmounted(() => {
  window.removeEventListener('wheel', handleWheel)
  window.removeEventListener('touchstart', handleTouchStart)
  window.removeEventListener('touchmove', handleTouchMove)
  window.removeEventListener('touchend', handleTouchEnd)
})

// 添加返回首页的方法
const goToHome = () => {
  router.push('/')
}
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* 添加过渡效果 */
select {
  appearance: none;
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3e%3cpath stroke='%23fb7299' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3e%3c/svg%3e");
  background-position: right 0.5rem center;
  background-repeat: no-repeat;
  background-size: 1.5em 1.5em;
  padding-right: 2.5rem;
}

select:focus {
  outline: none;
  box-shadow: 0 0 0 2px rgba(251, 114, 153, 0.2);
}

/* 移除深色模式媒体查询，始终使用浅色模式 */
</style>
