<template>
  <article class="flexcol sf-character-sheet" data-tourid="sheet">
    <!-- TODO: rm inline styles added to maintain consistent styling (required largely because of other inline styles) -->
    <!-- Header row -->
    <sf-characterheader />

    <!-- Main body row -->
    <div class="flexrow">
      <!-- Momentum on left -->
      <div
        class="flexcol margin-left nogrow"
        style="width: min-content"
        data-tourid="momentum"
      >
        <MomentumMeterSlider
          labelPosition="right"
          data-tooltip-direction="UP"
        />
      </div>

      <!-- Center area -->
      <div class="flexcol">
        <!-- Attributes -->
        <div
          class="flexrow stats"
          id="stats"
          style="margin-bottom: var(--ironsworn-spacer-xl)"
          data-tooltip-direction="UP"
          data-tourid="stats"
        >
          <attr-box attr="edge" />
          <attr-box attr="heart" />
          <attr-box attr="iron" />
          <attr-box attr="shadow" />
          <attr-box attr="wits" />
        </div>
        <TabSet
          :tabKeys="['legacies', 'assets', 'progress', 'connections', 'notes']"
          :id="`${data.actor._id}_sf-character-sheet`"
          data-tourid="tabs"
        >
          <TabList>
            <Tab tab-key="legacies" :text="$t('IRONSWORN.Legacies')" />
            <Tab tab-key="assets" :text="$t('IRONSWORN.ITEMS.TypeAsset')" />
            <Tab
              tab-key="progress"
              :text="$t('IRONSWORN.ITEMS.SubtypeProgress')"
            />
            <Tab
              tab-key="connections"
              :text="$t('IRONSWORN.ITEMS.SubtypeConnection')"
            />
            <Tab tab-key="notes" :text="$t('Notes')" />
          </TabList>
          <TabPanels>
            <TabPanel tab-key="legacies" class="flexcol">
              <SfLegacies />
            </TabPanel>
            <TabPanel tab-key="assets" class="flexcol">
              <SfAssets :class="$style.topPadding" />
            </TabPanel>
            <TabPanel tab-key="progress" class="flexcol">
              <SfProgresses :class="$style.topPadding" />
            </TabPanel>
            <TabPanel tab-key="connections" class="flexcol">
              <SfConnections :class="$style.topPadding" />
            </TabPanel>
            <TabPanel tab-key="notes" class="flexcol">
              <SfNotes />
            </TabPanel>
          </TabPanels>
        </TabSet>
      </div>

      <!-- Stats on right -->
      <PcConditionMeters
        class="flexcol margin-right"
        data-tooltip-direction="UP"
        labelPosition="left"
        data-tourid="resources"
      />
    </div>

    <!-- Impacts -->
    <hr class="nogrow" />
    <sf-impacts class="nogrow" data-tourid="impacts" />
  </article>
</template>

<style lang="less" module>
.topPadding {
  padding-top: var(--ironsworn-spacer-md);
}
</style>

<style lang="less">
.sf-character-sheet {
  gap: var(--ironsworn-spacer-lg);

  .stat-roll {
    text-transform: uppercase;
  }

  .condition-meters {
    .icon-button {
      flex-direction: column;

      .button-text {
        writing-mode: vertical-lr;
      }
    }
  }
}
</style>

<script lang="ts" setup>
import { computed, provide } from 'vue'
import AttrBox from './components/attr-box.vue'
import SfLegacies from './components/character-sheet-tabs/sf-legacies.vue'
import SfConnections from './components/character-sheet-tabs/sf-connections.vue'
import SfCharacterheader from './components/sf-characterheader.vue'
import SfImpacts from './components/sf-impacts.vue'
import SfAssets from './components/character-sheet-tabs/sf-assets.vue'
import SfProgresses from './components/character-sheet-tabs/sf-progresses.vue'
import SfNotes from './components/character-sheet-tabs/sf-notes.vue'
import { ActorKey } from './provisions.js'
import PcConditionMeters from './components/resource-meter/pc-condition-meters.vue'
import MomentumMeterSlider from './components/resource-meter/momentum-meter.vue'
import TabSet from './components/tabs/tab-set.vue'
import TabList from './components/tabs/tab-list.vue'
import Tab from './components/tabs/tab.vue'
import TabPanel from './components/tabs/tab-panel.vue'
import TabPanels from './components/tabs/tab-panels.vue'

const props = defineProps<{
  data: {
    actor: any
  }
}>()

provide(ActorKey, computed(() => props.data.actor) as any)
</script>
