<template>
  <div id="tags-view-container" class="tags-view-container" :key="forceRenderKey">
    <scroll-pane ref="scrollPaneRef" class="tags-view-wrapper" @scroll="handleScroll">
      <router-link
        v-for="tag in visitedViews"
        :key="tag.fullPath || tag.path"
        :data-path="tag.path"
        :class="isActive(tag) ? 'active' : ''"
        :to="{ path: tag.path, query: tag.query, fullPath: tag.fullPath }"
        class="tags-view-item"
        :style="activeStyle(tag)"
        @click.middle="!isAffix(tag) ? closeSelectedTag(tag) : ''"
        @contextmenu.prevent="openMenu(tag, $event)"
      >
        {{ tag.title }}
        <span v-if="!isAffix(tag)" @click.prevent.stop="closeSelectedTag(tag)">
          <close class="el-icon-close" style="width: 1em; height: 1em; vertical-align: middle" />
        </span>
      </router-link>
    </scroll-pane>
    <ul v-show="visible" :style="{ left: left + 'px', top: top + 'px' }" class="contextmenu">
      <li @click="refreshSelectedTag(selectedTag)"><refresh-right style="width: 1em; height: 1em" /> 刷新页面</li>
      <li v-if="!isAffix(selectedTag)" @click="closeSelectedTag(selectedTag)"><close style="width: 1em; height: 1em" /> 关闭当前</li>
      <li @click="closeOthersTags"><circle-close style="width: 1em; height: 1em" /> 关闭其他</li>
      <li v-if="!isFirstView()" @click="closeLeftTags"><back style="width: 1em; height: 1em" /> 关闭左侧</li>
      <li v-if="!isLastView()" @click="closeRightTags"><right style="width: 1em; height: 1em" /> 关闭右侧</li>
      <li @click="closeAllTags(selectedTag)"><circle-close style="width: 1em; height: 1em" /> 全部关闭</li>
    </ul>
  </div>
</template>

<script setup>
import ScrollPane from './ScrollPane';
import { getNormalPath } from '@/utils';
import useTagsViewStore from '@/store/modules/tagsView';
import useSettingsStore from '@/store/modules/settings';
import { sidebarRoutes } from '@/utils/routes';

const visible = ref(false);
const top = ref(0);
const left = ref(0);
const selectedTag = ref({});
const affixTags = ref([]);
const scrollPaneRef = ref(null);
const forceRenderKey = ref(0);
const isClosingTab = ref(false);

const nuxtApp = useNuxtApp();
const route = useRoute();
const router = useRouter();
const tagsViewStore = useTagsViewStore();

const visitedViews = computed(() => tagsViewStore.visitedViews);
const theme = computed(() => useSettingsStore().theme);
const routes = computed(() =>  sidebarRoutes);

watch(route, () => {
  if (isClosingTab.value) {
    setTimeout(() => {
      isClosingTab.value = false;
      addTags();
      moveToCurrentTag();
    }, 100);
  } else {
    addTags();
    moveToCurrentTag();
  }
});
watch(visible, (value) => {
  if (value) {
    document.body.addEventListener('click', closeMenu);
  } else {
    document.body.removeEventListener('click', closeMenu);
  }
});
onMounted(() => {
  initTags();
  addTags();
});

function isActive(r) {
  // 优先使用fullPath比较，确保query参数不同的页面能正确区分
  return r.fullPath === route.fullPath || (r.path === route.path && !r.fullPath);
}
function activeStyle(tag) {
  if (!isActive(tag)) return {};
  return {
    'background-color': theme.value,
    'border-color': theme.value
  };
}
function isAffix(tag) {
  return tag.meta && tag.meta.affix;
}
function isFirstView() {
  try {
    return selectedTag.value.fullPath === '/index' || selectedTag.value.fullPath === visitedViews.value[1].fullPath;
  } catch (err) {
    return false;
  }
}
function isLastView() {
  try {
    return selectedTag.value.fullPath === visitedViews.value[visitedViews.value.length - 1].fullPath;
  } catch (err) {
    return false;
  }
}
function filterAffixTags(routes, basePath = '') {
  let tags = [];
  routes.forEach((route) => {
    if (route.meta && route.meta.affix) {
      const tagPath = getNormalPath(basePath + '/' + route.path);
      tags.push({
        fullPath: tagPath,
        path: tagPath,
        name: route.name,
        meta: { ...route.meta }
      });
    }
    if (route.children) {
      const tempTags = filterAffixTags(route.children, route.path);
      if (tempTags.length >= 1) {
        tags = [...tags, ...tempTags];
      }
    }
  });
  return tags;
}
function initTags() {
  const res = filterAffixTags(routes.value);
  affixTags.value = res;
  for (const tag of res) {
    // Must have tag name
    if (tag.name) {
      useTagsViewStore().addVisitedView(tag);
    }
  }
}
function addTags() {
  const { name } = route;
  if (name) {
    useTagsViewStore().addView(route);
    if (route.meta.link) {
      useTagsViewStore().addIframeView(route);
    }
  }
  return false;
}
function moveToCurrentTag() {
  nextTick(() => {
    for (const r of visitedViews.value) {
      // 优先使用fullPath比较
      if (r.fullPath === route.fullPath || (r.path === route.path && !r.fullPath)) {
        scrollPaneRef.value?.moveToTarget(r);
        // when query is different then update
        if (r.fullPath !== route.fullPath) {
          useTagsViewStore().updateVisitedView(route);
        }
        break; // 找到匹配项后立即停止循环
      }
    }
  });
}
function refreshSelectedTag(view) {
  nuxtApp.$tab.refreshPage(view);
  if (route.meta.link) {
    useTagsViewStore().delIframeView(route);
  }
}
function closeSelectedTag(view) {
  // 立即关闭右键菜单
  closeMenu();
  
  // 设置关闭标志
  isClosingTab.value = true;
  
  // 直接调用store方法，避免通过plugin
  tagsViewStore.delView(view).then(({ visitedViews }) => {
    // 只有当关闭的是当前活跃标签时才需要跳转到其他页面
    if (isActive(view)) {
      toLastView(visitedViews, view);
    } else {
      // 如果关闭的不是当前标签，立即重置标志
      isClosingTab.value = false;
    }
    
    // 强制重新渲染
    forceRerender();
    
    // 等待DOM更新后再移动到当前标签
    nextTick(() => {
      if (scrollPaneRef.value && scrollPaneRef.value.moveToTarget) {
        moveToCurrentTag();
      }
    });
  }).catch((error) => {
    console.error('关闭标签页失败:', error);
    isClosingTab.value = false;
  });
}
function closeRightTags() {
  closeMenu();
  nuxtApp.$tab.closeRightPage(selectedTag.value).then((visitedViews) => {
    if (!visitedViews.find((i) => i.fullPath === route.fullPath)) {
      toLastView(visitedViews);
    }
    nextTick(() => {
      moveToCurrentTag();
    });
  });
}
function closeLeftTags() {
  closeMenu();
  nuxtApp.$tab.closeLeftPage(selectedTag.value).then((visitedViews) => {
    if (!visitedViews.find((i) => i.fullPath === route.fullPath)) {
      toLastView(visitedViews);
    }
    nextTick(() => {
      moveToCurrentTag();
    });
  });
}
function closeOthersTags() {
  closeMenu();
  router.push(selectedTag.value).catch(() => {});
  nuxtApp.$tab.closeOtherPage(selectedTag.value).then(() => {
    nextTick(() => {
      moveToCurrentTag();
    });
  });
}
function closeAllTags(view) {
  closeMenu();
  nuxtApp.$tab.closeAllPage().then(({ visitedViews }) => {
    if (affixTags.value.some((tag) => tag.path === route.path)) {
      return;
    }
    toLastView(visitedViews, view);
  });
}
function toLastView(visitedViews, view) {
  const latestView = visitedViews.slice(-1)[0];
  if (latestView) {
    router.push(latestView.fullPath).finally(() => {
      // 路由跳转完成后重置标志
      setTimeout(() => {
        isClosingTab.value = false;
      }, 50);
    });
  } else {
    // now the default is to redirect to the home page if there is no tags-view,
    // you can adjust it according to your needs.
    if (view.name === 'Dashboard') {
      // to reload home page
      router.replace({ path: '/redirect' + view.fullPath }).finally(() => {
        setTimeout(() => {
          isClosingTab.value = false;
        }, 50);
      });
    } else {
      router.push('/').finally(() => {
        setTimeout(() => {
          isClosingTab.value = false;
        }, 50);
      });
    }
  }
}
function openMenu(tag, e) {
  const menuMinWidth = 105;
  const container = document.getElementById('tags-view-container');
  const offsetLeft = container?.getBoundingClientRect().left || 0; // container margin left
  const offsetWidth = container?.offsetWidth || 0; // container width
  const maxLeft = offsetWidth - menuMinWidth; // left boundary
  const l = e.clientX - offsetLeft + 15; // 15: margin right

  if (l > maxLeft) {
    left.value = maxLeft;
  } else {
    left.value = l;
  }

  top.value = e.clientY;
  visible.value = true;
  selectedTag.value = tag;
}
function closeMenu() {
  visible.value = false;
}
function handleScroll() {
  closeMenu();
}
function forceRerender() {
  forceRenderKey.value += 1;
}
</script>

<style lang="scss" scoped>
.tags-view-container {
  height: 34px;
  width: 100%;
  background: #fff;
  border-bottom: 1px solid #d8dce5;
  box-shadow:
    0 1px 3px 0 rgba(0, 0, 0, 0.12),
    0 0 3px 0 rgba(0, 0, 0, 0.04);
  .tags-view-wrapper {
    .tags-view-item {
      display: inline-block;
      position: relative;
      cursor: pointer;
      height: 26px;
      line-height: 26px;
      border: 1px solid #d8dce5;
      color: #495060;
      background: #fff;
      padding: 0 8px;
      font-size: 12px;
      margin-left: 5px;
      margin-top: 4px;
      &:first-of-type {
        margin-left: 15px;
      }
      &:last-of-type {
        margin-right: 15px;
      }
      &.active {
        background-color: #42b983;
        color: #fff;
        border-color: #42b983;
        &::before {
          content: '';
          background: #fff;
          display: inline-block;
          width: 8px;
          height: 8px;
          border-radius: 50%;
          position: relative;
          margin-right: 5px;
        }
      }
    }
  }
  .contextmenu {
    margin: 0;
    background: #fff;
    z-index: 3000;
    position: absolute;
    list-style-type: none;
    padding: 5px 0;
    border-radius: 4px;
    font-size: 12px;
    font-weight: 400;
    color: #333;
    box-shadow: 2px 2px 3px 0 rgba(0, 0, 0, 0.3);
    li {
      margin: 0;
      padding: 7px 16px;
      cursor: pointer;
      &:hover {
        background: #eee;
      }
    }
  }
}
</style>

<style lang="scss">
//reset element css of el-icon-close
.tags-view-wrapper {
  .tags-view-item {
    .el-icon-close {
      width: 16px;
      height: 16px;
      vertical-align: 2px;
      border-radius: 50%;
      text-align: center;
      transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
      transform-origin: 100% 50%;
      &:before {
        transform: scale(0.6);
        display: inline-block;
        vertical-align: -3px;
      }
      &:hover {
        background-color: #b4bccc;
        color: #fff;
        width: 12px !important;
        height: 12px !important;
      }
    }
  }
}
</style>
