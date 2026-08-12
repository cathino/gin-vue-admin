<script setup>
import { computed, provide, toRef } from 'vue'
import HorizontalMenu from './HorizontalMenu.vue'
import MenuItem from './MenuItem.vue'
import { MENU_THEMES, findMenuPath, visibleItems } from './shared'
import { menuNavPad } from './variants'

defineOptions({ name: 'GvaMenu' })

const props = defineProps({
  items: { type: Array, default: () => [] },
  orientation: {
    type: String,
    default: 'vertical',
    validator: (v) => ['vertical', 'horizontal'].includes(v)
  },
  theme: {
    type: String,
    default: 'design',
    validator: (v) => MENU_THEMES.includes(v)
  },
  collapsed: { type: Boolean, default: false },
  // 折叠态是否在图标下方展示标题（小字号）
  collapsedShowTitle: { type: Boolean, default: false },
  active: { type: String, default: '' },
  itemHeight: { type: Number, default: 48 },
  class: { type: null, default: '' }
})
const emit = defineEmits(['select'])
// 展开的分支 key 列表；上层用 v-model:open-keys 双向绑定（激活路径/分组默认展开由 useMenuActive 维护）
const openKeys = defineModel('openKeys', { type: Array, default: () => [] })

const topItems = computed(() => visibleItems(props.items))
// 侧栏底色由外层布局容器负责铺满整栏；nav 保持透明。
// 左右内边距唯一由菜单风格决定（与收缩与否无关）：design 全宽贴边，light / group 留白圆角。
const navPadClass = computed(() => menuNavPad(props.theme))

const select = (key) => emit('select', key)

const collectKeys = (node) => {
  const keys = [node.name]
  const children = node.children || []
  children.forEach((c) => keys.push(...collectKeys(c)))
  return keys
}
// 展开策略：group 自由多开；其余风格 unique-opened（只保留当前路径，收起同级）
const toggle = (node) => {
  const opened = openKeys.value.includes(node.name)
  if (props.theme === 'group') {
    openKeys.value = opened
      ? openKeys.value.filter((k) => k !== node.name)
      : [...openKeys.value, node.name]
    return
  }
  if (opened) {
    const sub = collectKeys(node)
    openKeys.value = openKeys.value.filter((k) => !sub.includes(k))
  } else {
    openKeys.value = findMenuPath(props.items, node.name)
  }
}

provide('gvaMenuCtx', {
  theme: toRef(props, 'theme'),
  collapsed: toRef(props, 'collapsed'),
  collapsedShowTitle: toRef(props, 'collapsedShowTitle'),
  active: toRef(props, 'active'),
  itemHeight: toRef(props, 'itemHeight'),
  openKeys,
  select,
  toggle
})
</script>

<template>
  <HorizontalMenu
    v-if="orientation === 'horizontal'"
    :items="topItems"
    :active="active"
    :class="props.class"
    @select="select"
  />
  <nav v-else :class="['flex flex-col gap-0.5', navPadClass, props.class]">
    <MenuItem v-for="item in topItems" :key="item.name" :node="item" :depth="0" />
  </nav>
</template>
