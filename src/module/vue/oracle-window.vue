<template>
  <div class="flexcol">
    <TabSet
      :tabKeys="['ironsworn', 'starforged']"
      id="oracleset"
      :default-key="defaultTab"
    >
      <TabList>
        <Tab tab-key="ironsworn" text="Ironsworn" />
        <Tab tab-key="starforged" text="Starforged" />
      </TabList>
      <TabPanels>
        <TabPanel tab-key="ironsworn" class="flexcol">
          <Suspense>
            <SfMovesheetoracles toolset="ironsworn" />
          </Suspense>
        </TabPanel>
        <TabPanel tab-key="starforged" class="flexcol">
          <Suspense>
            <SfMovesheetoracles toolset="starforged" />
          </Suspense>
        </TabPanel>
      </TabPanels>
    </TabSet>
  </div>
</template>

<script setup lang="ts">
import { IronswornSettings } from '../helpers/settings'
import SfMovesheetoracles from './components/sf-movesheetoracles.vue'
import TabList from './components/tabs/tab-list.vue'
import TabPanel from './components/tabs/tab-panel.vue'
import TabPanels from './components/tabs/tab-panels.vue'
import TabSet from './components/tabs/tab-set.vue'
import Tab from './components/tabs/tab.vue'

const defaultTab = IronswornSettings.starforgedToolsEnabled
  ? 'starforged'
  : 'ironsworn'
</script>
