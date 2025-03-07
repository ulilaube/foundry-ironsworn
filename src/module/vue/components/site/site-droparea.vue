<template>
  <DropTarget is="div" class="flexcol box" :dropType="itemType">
    <div
      v-if="item"
      style="position: relative; padding: var(--ironsworn-spacer-xl)"
    >
      <div class="flexrow">
        <document-img
          :document="item"
          size="40px"
          class="nogrow"
          style="margin-right: var(--ironsworn-spacer-md)"
        />

        <div class="flexcol">
          <h4 style="margin: 0">{{ item.name }}</h4>
          <p>{{ item.system.summary }}</p>
        </div>

        <div
          class="flexrow"
          v-if="editMode"
          style="
            position: absolute;
            top: var(--ironsworn-spacer-md);
            right: var(--ironsworn-spacer-md);
          "
        >
          <IronBtn block icon="fa:trash" @click="destroy" />
          <IronBtn block icon="fa:pen-to-square" @click="edit" />
        </div>
      </div>
    </div>

    <div v-else style="padding: 1em; width: 100%">
      <div class="flexcol">
        <h4 style="margin: 0">{{ $t(titleKey) }}</h4>
        <btn-compendium
          :compendium="compendiumKey"
          style="padding: 0 2em"
          class="inset"
          block
          :text="$t('IRONSWORN.OpenCompendium')"
        />
      </div>
    </div>
  </DropTarget>
</template>

<script setup lang="ts">
import { inject, Ref } from '@vue/runtime-core'
import { $ActorKey, ActorKey } from '../../provisions'
import { computed } from 'vue'
import DocumentImg from '../document-img.vue'
import IronBtn from '../buttons/iron-btn.vue'
import BtnCompendium from '../buttons/btn-compendium.vue'
import DropTarget from '../../drop-target.vue'

const props = defineProps<{
  item: any
  itemType: string
  titleKey: string
  compendiumKey: string
}>()

const $actor = inject($ActorKey)

const actor = inject(ActorKey) as Ref
const editMode = computed(() => {
  return actor.value.flags['foundry-ironsworn']?.['edit-mode']
})

function foundryitem() {
  return props.item && $actor?.items.get(props.item._id)
}

function destroy() {
  Dialog.confirm({
    title: game.i18n.format('DOCUMENT.Delete', {
      type: game.i18n.localize('DOCUMENT.Item'),
    }),
    content: `<p><strong>${game.i18n.localize(
      'IRONSWORN.ConfirmDelete'
    )}</strong></p>`,
    yes: () => foundryitem()?.delete(),
    defaultYes: false,
  })
}

function edit() {
  foundryitem()?.sheet?.render(true)
}
</script>
