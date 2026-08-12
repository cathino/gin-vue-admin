<template>
  <div class="gva-theme-font">
    <!-- 布局模式 -->
    <div class="mb-6">
      <div class="gva-theme-section-header">
        <span class="gva-theme-section-title">布局模式</span>
      </div>
      <div class="gva-theme-section-content">
        <LayoutModeCard v-model="settings.layout.mode" />
      </div>
    </div>

    <!-- 菜单风格 -->
    <div class="mb-6">
      <div class="gva-theme-section-header">
        <span class="gva-theme-section-title">菜单风格</span>
      </div>
      <div class="gva-theme-section-content">
        <MenuThemeSelector v-model="settings.menu.theme" />
        <div v-if="showDarkSider" class="gva-theme-card-bg mt-3">
          <SettingItem label="深色侧边栏">
            <g-switch v-model="settings.menu.darkSider" aria-label="深色侧边栏" />
          </SettingItem>
        </div>
      </div>
    </div>

    <!-- 顶栏：可见性 + 配色 -->
    <div class="mb-6">
      <div class="gva-theme-section-header">
        <span class="gva-theme-section-title">顶栏</span>
      </div>
      <div class="gva-theme-section-content">
        <div class="gva-theme-card-bg">
          <SettingItem label="显示面包屑">
            <g-switch v-model="settings.header.breadcrumb.visible" aria-label="显示面包屑" />
          </SettingItem>
          <SettingItem label="显示面包屑图标">
            <g-switch
              v-model="settings.header.breadcrumb.showIcon"
              :disabled="!settings.header.breadcrumb.visible"
              aria-label="显示面包屑图标"
            />
          </SettingItem>
          <SettingItem label="显示刷新按钮">
            <g-switch v-model="settings.header.refresh.visible" aria-label="显示刷新按钮" />
          </SettingItem>
          <SettingItem label="显示搜索按钮">
            <g-switch v-model="settings.header.search.visible" aria-label="显示搜索按钮" />
          </SettingItem>
          <SettingItem label="显示折叠按钮">
            <g-switch v-model="settings.header.collapseButton.visible" aria-label="显示折叠按钮" />
          </SettingItem>
          <SettingItem label="顶栏背景">
            <template #suffix>
              <span class="text-xs text-muted-foreground ml-2">留空跟随主题</span>
            </template>
            <g-color-picker
              v-model="settings.header.bg"
              alpha
              clearable
              format="rgb"
              placeholder="跟随主题"
              aria-label="顶栏背景"
            />
          </SettingItem>
          <SettingItem label="顶栏阴影">
            <g-select
              v-model="settings.header.shadow"
              class="min-w-24"
              :options="[
                { label: '无', value: 'none' },
                { label: '小', value: 'sm' },
                { label: '中', value: 'md' },
                { label: '大', value: 'lg' }
              ]"
            />
          </SettingItem>
          <SettingItem label="标签栏背景">
            <g-color-picker
              v-model="settings.tab.bg"
              alpha
              clearable
              format="rgb"
              placeholder="跟随主题"
              aria-label="标签栏背景"
            />
          </SettingItem>
          <SettingItem label="标签栏阴影">
            <g-select
              v-model="settings.tab.shadow"
              class="min-w-24"
              :options="[
                { label: '无', value: 'none' },
                { label: '小', value: 'sm' },
                { label: '中', value: 'md' },
                { label: '大', value: 'lg' }
              ]"
            />
          </SettingItem>
          <div class="flex items-center gap-1.5 py-2.5 text-xs text-muted-foreground leading-snug">
            <svg-icon icon="lucide:info" class="flex-shrink-0" />
            <span>暗色模式下将基于以上配色自动推导深色版本，无需单独设置</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 界面 -->
    <div class="mb-6">
      <div class="gva-theme-section-header">
        <span class="gva-theme-section-title">界面</span>
      </div>
      <div class="gva-theme-section-content">
        <div class="gva-theme-card-bg">
          <SettingItem label="显示标签页">
            <g-switch v-model="settings.tab.visible" aria-label="显示标签页" />
          </SettingItem>
          <SettingItem label="页面切换动画">
            <g-select
              v-model="settings.page.transition"
              class="min-w-24"
              :options="[
                { label: '淡入淡出', value: 'fade' },
                { label: '滑动', value: 'slide' },
                { label: '缩放', value: 'zoom' },
                { label: '无动画', value: 'none' }
              ]"
            />
          </SettingItem>
        </div>
      </div>
    </div>

    <!-- 侧栏配置 -->
    <div class="mb-6">
      <div class="gva-theme-section-header">
        <span class="gva-theme-section-title">侧栏配置</span>
      </div>
      <div class="gva-theme-section-content">
        <div class="gva-theme-card-bg">
          <SettingItem label="展开宽度">
            <g-number-field v-model="settings.layout.sideWidth" :min="150" :max="400" :step="10" class="w-28" />
          </SettingItem>
          <SettingItem label="收缩宽度">
            <g-number-field v-model="settings.layout.sideCollapsedWidth" :min="60" :max="100" class="w-28" />
          </SettingItem>
          <SettingItem label="菜单项高度">
            <g-number-field v-model="settings.layout.sideItemHeight" :min="30" :max="50" class="w-28" />
          </SettingItem>
          <SettingItem label="收起时是否显示标题">
            <g-switch v-model="settings.layout.showTitleOnCollapse" aria-label="收起时是否显示标题" />
          </SettingItem>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { storeToRefs } from 'pinia'
import { useAppStore, useThemeStore } from '@/pinia'
import { useLayoutMode } from '@/hooks/useLayoutMode'
import LayoutModeCard from '../../components/layoutModeCard.vue'
import MenuThemeSelector from '../../components/menuThemeSelector.vue'
import SettingItem from '../../components/settingItem.vue'

defineOptions({
  name: 'LayoutSettings'
})

const themeStore = useThemeStore()
const appStore = useAppStore()
const { settings, darkMode } = storeToRefs(themeStore)
const { device } = storeToRefs(appStore)
const { effectiveMode } = useLayoutMode()

// 「深色侧边栏」仅在浅色主题、且屏幕上确有侧栏（含移动抽屉、平板强制通栏）时可配；
// 全局暗色或顶部导航（桌面 head）下无意义，隐藏。
const showDarkSider = computed(
  () => !darkMode.value && (device.value === 'mobile' || effectiveMode.value !== 'head')
)
</script>
