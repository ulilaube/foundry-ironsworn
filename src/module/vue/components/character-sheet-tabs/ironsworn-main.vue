<template>
  <div class="flexrow">
    <div class="flexcol">
      <section class="sheet-area flexcol">
        <!-- Bonds -->
        <Bonds :compactProgress="true" data-tourid="bonds" />

        <hr class="nogrow" />
        <!-- Assets -->
        <DropTarget
          is="div"
          dropType="asset"
          class="flexcol item-list"
          data-tourid="assets"
        >
          <h4 class="nogrow">{{ $t('IRONSWORN.ITEMS.TypeAsset') }}</h4>

          <CollapseTransition
            tag="div"
            class="nogrow flexcol"
            group
            :class="$style.items"
          >
            <div class="flexrow" v-for="(asset, i) in assets" :key="asset._id">
              <OrderButtons
                v-if="editMode"
                :i="i"
                :length="assets.length"
                @sortUp="assetSortUp"
                @sortDown="assetSortDown"
              />
              <Asset :asset="asset" class="item-row" />
            </div>
          </CollapseTransition>
          <div class="flexrow nogrow" style="text-align: center">
            <IronBtn
              icon="fa:book-atlas"
              @click="assetBrowser"
              block
              :text="$t('IRONSWORN.ITEMS.TypeAsset')"
            />
          </div>
        </DropTarget>
      </section>
    </div>
    <ActiveCompletedProgresses
      :compactProgress="true"
      :class="$style.progress"
      data-tourid="progress"
    />
  </div>
</template>
<style lang="less" module>
.items {
  gap: var(--ironsworn-spacer-md);
}

.progress {
  margin-top: var(--ironsworn-spacer-md);
}
</style>
<style lang="less" scoped>
h4 {
  text-transform: uppercase;
}

h3 {
  transition: background-color 0.2s ease;
  margin: var(--ironsworn-spacer-md) 0;

  i {
    width: 15px;
    text-align: center;
  }
}
</style>

<script lang="ts" setup>
import { computed, inject, reactive, Ref } from 'vue'
import { $ActorKey, ActorKey } from '../../provisions'
import Bonds from '../bonds.vue'
import OrderButtons from '../order-buttons.vue'
import Asset from '../asset/asset.vue'
import IronBtn from '../buttons/iron-btn.vue'
import ActiveCompletedProgresses from '../active-completed-progresses.vue'
import { AssetCompendiumBrowser } from '../../../item/asset-compendium-browser'
import CollapseTransition from '../transition/collapse-transition.vue'
import DropTarget from '../../drop-target.vue'

const actor = inject(ActorKey) as Ref
const $actor = inject($ActorKey)

const assets = computed(() => {
  return actor.value?.items
    .filter((x) => x.type === 'asset')
    .sort((a, b) => (a.sort || 0) - (b.sort || 0))
})
const editMode = computed(() => {
  return actor.value?.flags['foundry-ironsworn']?.['edit-mode']
})

async function applySort(oldI, newI, sortBefore, collection) {
  const sorted = collection.sort(
    (a, b) => (a.data.sort || 0) - (b.data.sort || 0)
  )
  const updates = SortingHelpers.performIntegerSort(sorted[oldI], {
    target: sorted[newI],
    siblings: sorted,
    sortBefore,
  })
  await Promise.all(updates.map(({ target, update }) => target.update(update)))
}
function assetSortUp(i) {
  const items = $actor?.items.filter((x) => x.type === 'asset')
  applySort(i, i - 1, true, items)
}
function assetSortDown(i) {
  const items = $actor?.items.filter((x) => x.type === 'asset')
  applySort(i, i + 1, false, items)
}

let theAssetBrowser: AssetCompendiumBrowser | undefined
function assetBrowser() {
  if (!theAssetBrowser) {
    theAssetBrowser = new AssetCompendiumBrowser(
      $actor?.toolset ?? 'starforged'
    )
  }
  theAssetBrowser.render(true, { focus: true })
}
</script>
