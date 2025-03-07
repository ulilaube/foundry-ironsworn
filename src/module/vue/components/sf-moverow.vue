<template>
  <Collapsible
    v-bind="$props.collapsible"
    class="movesheet-row"
    :class="$style.sfMoveRow"
    data-tooltip-direction="LEFT"
    :baseId="`move_row_${move.moveItem().id}`"
    ref="$collapsible"
    :contentWrapperClass="$style.contentWrapper"
    :toggleWrapperIs="`h${headingLevel}`"
    :toggleSectionClass="[$style.toggleSection, toggleSectionClass]"
    :icon="null"
    :toggleButtonClass="['bordered', $style.toggleButton, toggleButtonClass]"
    :toggleTooltip="toggleTooltip"
    :toggleWrapperClass="$style.toggleWrapper"
    :toggleLabel="move?.displayName"
    :data-move-id="move.moveItem().id"
    :data-move-uuid="move.moveItem().uuid"
  >
    <template #after-toggle>
      <section
        :class="$style.moveControls"
        class="nogrow"
        data-tooltip-direction="UP"
        data-tourid="move-buttons"
      >
        <BtnRollmove
          :disabled="!canRoll"
          :move="move"
          :class="$style.moveButton"
          :override-click="onRollClick !== undefined"
          @click="$emit('rollClick')"
        />
        <BtnOracle
          :node="data.oracles[0] ?? {}"
          :disabled="preventOracle"
          :class="$style.moveButton"
          :override-click="onOracleClick !== undefined"
          @click="$emit('oracleClick')"
        />
        <BtnSendmovetochat :move="move" :class="$style.moveButton" />
      </section>
    </template>
    <template #default>
      <RulesTextMove
        @moveclick="moveClick"
        :move="move"
        :class="$style.moveSummary"
      >
        <template #after-footer>
          <OracleTreeNode
            :class="$style.moveOracle"
            v-for="node of data.oracles"
            :key="node.displayName"
            :node="node"
          />
        </template>
      </RulesTextMove>
    </template>
  </Collapsible>
</template>

<style lang="less" module>
@import (reference) '../../../styles/mixins.less';

.thematicColorMixin {
  --ironsworn-color-thematic: v-bind('thematicColor');

  border-color: var(--ironsworn-color-thematic);
  background-color: var(--ironsworn-color-thematic);
  color: var(--ironsworn-color-light);
}

.cardColorsMixin {
  border-color: var(--ironsworn-color-thematic);
  background-color: var(--ironsworn-color-bg);
  color: var(--ironsworn-color-fg);
}

.sfMoveRow {
  --ironsworn-line-height: (--ironsworn-line-height-md);
  .thematicColorMixin();

  position: relative;
  padding: 0 var(--ironsworn-spacer-md);

  &[aria-expanded='true'] {
    padding-top: var(--ironsworn-spacer-md);
    padding-bottom: var(--ironsworn-spacer-md);
  }
}

.moveSummary {
  padding: var(--ironsworn-spacer-lg) var(--ironsworn-spacer-lg)
    var(--ironsworn-spacer-md);
}

.moveButton {
  --ironsworn-color-clickable-text: var(--ironsworn-color-light);
  --ironsworn-color-clickable-text-hover: var(--ironsworn-color-light-warm);
  .clickableTextMixin();

  align-self: center;
  font-size: var(--font-size-20);
  aspect-ratio: 1 !important;
}

.toggleButton {
  --ironsworn-color-clickable-text: var(--ironsworn-color-light);
  --ironsworn-color-clickable-text-hover: var(--ironsworn-color-light-warm);

  .clickableTextMixin();
  .textStrokeMixin( var(--ironsworn-color-dark));
  .thematicColorMixin();

  display: flex;
  flex-direction: row;
  align-items: center;
  border: none;
  border-width: var(--ironsworn-border-width-md)
    var(--ironsworn-border-width-md) 0 var(--ironsworn-border-width-md);
  border-style: solid;
  border-color: transparent;
  background: none;
  padding: 0;
  padding-left: var(--ironsworn-spacer-sm);
  height: 100%;
  text-align: left;
  font-size: var(--font-size-16);

  &:hover {
    box-shadow: none;
  }
}

.contentWrapper {
  border: var(--ironsworn-border-width-md) solid var(--ironsworn-color-light);
  border-radius: 0 var(--ironsworn-border-radius-lg)
    var(--ironsworn-border-radius-lg) var(--ironsworn-border-radius-lg);
  background-color: var(--ironsworn-color-bg-80);
  color: var(--ironsworn-color-fg);
}

.moveControls {
  display: flex;
  flex-flow: row;
  background: none;
}

.toggleSection {
  display: flex;
  flex-flow: row nowrap;
  gap: var(--ironsworn-spacer-md);
}

.toggleWrapper {
  transition: var(--ironsworn-transition);
  border: var(--ironsworn-border-width-md) solid transparent;
  border-bottom-width: 0;
  border-top-left-radius: var(--ironsworn-border-radius-lg);
  border-top-right-radius: var(--ironsworn-border-radius-lg);
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
  line-height: 1.5;

  header:not(:last-child) & {
    border-color: var(--ironsworn-color-clickable-block-border-selected);
    background-color: var(--ironsworn-color-dark);
    color: var(--ironsworn-color-light);
  }
}

.moveOracle {
  border-width: var(--ironsworn-border-width-md);
  border-style: solid;
  border-radius: var(--ironsworn-border-radius-sm);
  border-color: var(--ironsworn-color-border);
  padding: 0;

  h4 {
    font-size: var(--font-size-16);

    button.icon-button {
      height: inherit;
    }
  }
}
</style>

<script setup lang="ts">
import { computed, ExtractPropTypes, provide, reactive, ref } from 'vue'
import { getDFOracleByDfId } from '../../dataforged'
import { Move } from '../../features/custommoves'
import { IOracleTreeNode, walkOracle } from '../../features/customoracles'
import { IronswornItem } from '../../item/item'
import { moveHasRollableOptions } from '../../rolls/preroll-dialog'
import BtnRollmove from './buttons/btn-rollmove.vue'
import BtnSendmovetochat from './buttons/btn-sendmovetochat.vue'
import OracleTreeNode from './oracle-tree-node.vue'
import RulesTextMove from './rules-text/rules-text-move.vue'
import Collapsible from './collapsible/collapsible.vue'
import BtnOracle from './buttons/btn-oracle.vue'
import { ItemKey, $ItemKey } from '../provisions.js'
import { enrichMarkdown } from '../vue-plugin.js'
import { SFMoveDataPropertiesData } from '../../item/itemtypes'
import { uniq } from 'lodash'

const props = withDefaults(
  defineProps<{
    move: Move
    headingLevel?: number
    thematicColor?: string | null
    toggleSectionClass?: any
    toggleButtonClass?: any
    oracleDisabled?: true | false | null

    // Hack: if we declare `click` in the emits, there's no $attrs['onClick']
    // This allows us to check for presence and still use $emit('click')
    // https://github.com/vuejs/core/issues/4736#issuecomment-934156497
    onRollClick?: Function
    onOracleClick?: Function
    /**
     * Props to be passed to the Collapsible component.
     */
    collapsible?: Omit<
      ExtractPropTypes<typeof Collapsible>,
      | 'contentWrapperClass'
      | 'toggleWrapperIs'
      | 'toggleSectionClass'
      | 'noIcon'
      | 'toggleButtonClass'
      | 'toggleTooltip'
      | 'toggleWrapperClass'
      | 'toggleLabel'
    >
  }>(),
  {
    headingLevel: 4,
    toggleSectionClass: '',
    toggleButtonClass: '',
    oracleDisabled: null,
  }
)

const $item = computed(() => props.move.moveItem() as IronswornItem)
const $itemSystem = computed(
  () => $item.value?.system as SFMoveDataPropertiesData
)

provide(ItemKey, computed(() => $item.value.toObject()) as any)
provide($ItemKey, $item.value)

const data = reactive({
  oracles: [] as IOracleTreeNode[],
})

const $collapsible = ref<typeof Collapsible>()

type CollapsibleEmits = (typeof Collapsible)['$emit']

interface MoveRowEmits extends CollapsibleEmits {
  rollClick(): void
  oracleClick(): void
}

const $emit = defineEmits<MoveRowEmits>()

const canRoll = computed(() => {
  if (props.onRollClick) return true
  return moveHasRollableOptions($item.value)
})
const preventOracle = computed(() => {
  if (props.oracleDisabled !== null) return props.oracleDisabled
  return data.oracles.length !== 1
})

const toggleTooltip = computed(() =>
  // @ts-ignore
  enrichMarkdown($item.value.system.Trigger?.Text)
)

const moveId = computed(() => props.move.moveItem().id)

const oracleIds = uniq([
  ...($itemSystem.value?.Oracles ?? []),
  ...(props.move.dataforgedMove?.Oracles ?? []),
])
Promise.all(oracleIds.map(getDFOracleByDfId)).then(async (dfOracles) => {
  const nodes = await Promise.all(dfOracles.map(walkOracle))
  data.oracles.push(...nodes)
})

// Outbound link clicks: broadcast events
function moveClick(move: IronswornItem) {
  CONFIG.IRONSWORN.emitter.emit('highlightMove', move.uuid)
}

defineExpose({
  moveId: moveId.value,
  $collapsible,
})
</script>
