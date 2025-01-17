<script lang="ts" setup>
import { computed, ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { NConfigProvider, NMessageProvider, NDialogProvider, darkTheme, lightTheme, NElement, NLoadingBarProvider } from 'naive-ui';
import LeftDrawer from './components/LeftDrawer/LeftDrawer.vue';
import DrawerComponent from './components/LeftDrawer/DrawerComponent.vue';
import { useAppStore } from './stores/AppStore';
import ConfettiCanvasProvider from './components/confetti/ConfettiCanvasProvider.vue';
import SettingsConfigProvider from './components/settings/SettingsConfigProvider.vue';
import SearchProvider from './components/search/SearchProvider.vue';
import BeforeStartModal from './components/BeforeStartModal.vue';
import BugReport from './components/debug/BugReport.vue';

const appStore = useAppStore();
const router = useRouter();

darkTheme.Button!!.common!!.errorColor = "#ED4245";
darkTheme.Button!!.common!!.infoColor = "#62CDFE";

const collapsed = computed(() => appStore.leftDrawer.collapsed);
const theme = computed(() => appStore.theme);
const leftDrawer = computed(() => appStore.leftDrawer)
const search = computed(() => appStore.search)
const currentRouteName = ref("")
appStore.load();
appStore.startRealtime();

onMounted(() => {
  currentRouteName.value = router.currentRoute.value.name as string
  router.afterEach((to, from, failure) => {
    currentRouteName.value = to.name as string;
  })
})
</script>

<template>
  <div class="lightable-red">
    <ConfettiCanvasProvider>
      <NConfigProvider :theme="(theme == 'Dark') ? darkTheme : lightTheme">

        <BeforeStartModal />
        <BugReport/>
        <NLoadingBarProvider>
          <NDialogProvider>
            <NElement>
              <div id="profile-modal"></div>
              <NMessageProvider>
                <SettingsConfigProvider>
                  <SearchProvider :show="search.show">
                    <div class="general-co">
                      <div class="lightable-drawer" v-if="leftDrawer.show">
                        <LeftDrawer>
                          <template #top>
                            <DrawerComponent :pair="item" v-for="(item, index) in leftDrawer.components" v-bind:key="index" />
                          </template>
                          <template #groups>
                            <div class="group" v-for="(group, index) in leftDrawer.groups" v-bind:key="index">
                              <div class="header ns">
                                <span>{{ group.name }}</span>
                              </div>
                              <div class="group-contents" :id="`group-${group.name}`">
                                <div class="group-item" v-for="(gitem, gid) in group.items" v-bind:key="gid" :id="`group-item-${gitem.text}`">
                                  <DrawerComponent :pair="gitem" :style="{ width: '88%' }" />
                                </div>
                              </div>
                            </div>
                          </template>
                        </LeftDrawer>
                      </div>
                      <div class="page" :style="{ 'background': `${theme === 'Dark' ? 'var(--lightable-dark-bg)' : 'var(--lightable-light-bg)'}` }">
                        <div class="content" :style="collapsed ||  currentRouteName.match(/accounts|signup|invite|login/) ? 'width: 100vw;' : 'width: calc(100vw - 250px)'">
                          <router-view />
                        </div>
                      </div>
                    </div>
                  </SearchProvider>
                </SettingsConfigProvider>
              </NMessageProvider>
            </NElement>
          </NDialogProvider>
        </NLoadingBarProvider>
      </NConfigProvider>
    </ConfettiCanvasProvider>
  </div>
</template>

<style lang="scss" scoped>
.lightable-red {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  height: 100vh;

  .general-co {
    height: 100vh;
    width: 100%;
    display: flex;
    flex-direction: row;

    .page {
      width: 100%;
      // background-color: #121219;
      display: flex;
      flex-direction: column;
      justify-content: flex-start;

      .content {
        width: calc(100vw - 250px);
        height: 100vh;
        transition: all 250ms ease;
      }
    }
  }
}

.group {
  width: 100%;

  .header {
    margin-left: 10px;
  }

  .group-contents {
    display: flex;
    flex-direction: column;
    width: 100%;
    justify-content: center;
    align-items: flex-start;
    margin-left: 8px;

    .group-item {
      width: 100%;
    }
  }
}
</style>
