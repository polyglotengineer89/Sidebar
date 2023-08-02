<template>
  <div ref="sidebarMenuRef" :class="[sidebarClass]" :style="{ 'max-width': sidebarWidth }">
    <span v-if="!hideHeader">
      <slot name="header" />
    </span>
  
    <sidebar-menu-scroll>
      <ul class="vsm--menu" :style="{ width: sidebarWidth }">
        <sidebar-menu-item v-for="item in computedMenu" :key="item.id" :curr-role="currRole" :item="item">
          <template #dropdown-icon="{ isOpen }">
            <slot name="dropdown-icon" v-bind="{ isOpen }">
              <span class="vsm--arrow_default" />
            </slot>
          </template>
        </sidebar-menu-item>
      </ul>
    </sidebar-menu-scroll>
    <slot name="footer" />
    <button v-if="!hideToggle" class="vsm--toggle-btn" @click="onToggleClick">
      <slot name="toggle-icon">
        <span class="vsm--toggle-btn_default" />
      </slot>
    </button>
  </div>
</template>

<script>
import {
  watch,
  ref,
  getCurrentInstance,
  onMounted,
  onUnmounted,
  computed,
} from 'vue'
import { initSidebar } from '../use/useSidebar'

import SidebarMenuItem from './SidebarMenuItem.vue'
import SidebarMenuScroll from './SidebarMenuScroll.vue'

export default {
  compatConfig: { MODE: 3 },
  name: 'SidebarMenu',
  components: {
    SidebarMenuItem,
    SidebarMenuScroll,
  },
  props: {
    menu: {
      type: Array,
      required: true,
    },
    currRole: {
      type: String,
      default: '',
      required: true,
    },
    collapsed: {
      type: Boolean,
      default: false,
    },
    roles: {
      type: Array,
      default: undefined,
    },
    width: {
      type: String,
      default: '200px',
    },
    widthCollapsed: {
      type: String,
      default: '44px',
    },
    showChild: {
      type: Boolean,
      default: false,
    },
    theme: {
      type: String,
      default: undefined,
      validator: (value) => ['', 'white-theme'].includes(value),
    },
    showOneChild: {
      type: Boolean,
      default: false,
    },
    rtl: {
      type: Boolean,
      default: false,
    },
    relative: {
      type: Boolean,
      default: false,
    },
    hideToggle: {
      type: Boolean,
      default: false,
    },
    disableHover: {
      type: Boolean,
      default: false,
    },
    linkComponentName: {
      type: String,
      default: undefined,
    },
  },
  emits: {
    'item-click'(event, item) {
      return !!(event && item)
    },
    'update:collapsed'(collapsed) {
      return !!(typeof collapsed === 'boolean')
    },
  },
  setup(props, context) {
    const {
      getSidebarRef: sidebarMenuRef,
      getIsCollapsed: isCollapsed,
      updateIsCollapsed,
      unsetMobileItem,
      updateCurrentRoute,
    } = initSidebar(props, context)

    const hideHeader = ref(false)

    const computedMenu = computed(() => {
      let id = 0
      function transformItems(items) {
        function randomId() {
          return `${Date.now() + '' + id++}`
        }
        return items.map((item) => {
          return {
            id: randomId(),
            ...item,
            ...(item.child && { child: transformItems(item.child) }),
          }
        })
      }
      return transformItems(props.menu)
    })

    const sidebarWidth = computed(() => {
      return isCollapsed.value ? props.widthCollapsed : props.width
    })

    const sidebarClass = computed(() => {
      return [
        'v-sidebar-menu',
        !isCollapsed.value ? 'vsm_expanded' : 'vsm_collapsed',
        props.theme && `vsm_${props.theme}`,
        props.rtl && 'vsm_rtl',
        props.relative && 'vsm_relative',
      ]
    })

    const onToggleClick = () => {
      hideHeader.value = !hideHeader.value;
      unsetMobileItem()
      updateIsCollapsed(!isCollapsed.value)
      context.emit('update:collapsed', isCollapsed.value)
    }

    watch(
      () => props.collapsed,
      (currentCollapsed) => {
        unsetMobileItem()
        updateIsCollapsed(currentCollapsed)
      }
    )

    const router =
      getCurrentInstance().appContext.config.globalProperties.$router
    if (!router) {
      onMounted(() => {

        window.addEventListener('hashchange', updateCurrentRoute)
      })
      onUnmounted(() => {
        window.removeEventListener('hashchange', updateCurrentRoute)
      })
    }

    return {
      hideHeader,
      sidebarMenuRef,
      isCollapsed,
      computedMenu,
      sidebarWidth,
      sidebarClass,
      onToggleClick,
      onRouteChange: updateCurrentRoute,
    }
  },
}
</script>

<style lang="scss">
@import '../scss/vue-sidebar-menu';

.v-sidebar-menu {
  background: #3c4b64;
  color: #fff;
}

.v-sidebar-menu .vsm--link_level-1 .vsm--icon {
  background-color: transparent;
}

.v-sidebar-menu .vsm--badge_default,
.v-sidebar-menu .vsm--toggle-btn {
  background-color: rgba(0, 0, 21, 0.2);
}

.v-sidebar-menu .vsm--link{
  padding: 11px 5px;
}

body{
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto,
    Helvetica Neue, Arial, Noto Sans, sans-serif, Apple Color Emoji,
    Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;
  font-size: 0.875rem;
  font-weight: 400;
  line-height: 1.5;
}
</style>
