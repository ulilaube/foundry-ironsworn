<template>
  <div
    class="box flexcol block"
    :class="{
      [$style.wrapper]: true,
      [$style.interactive]: !isMomentum,
      'isiconbg-d10-tilt': !isMomentum,
    }"
  >
    <h4 :data-tooltip="tooltip" @click="click">{{ i18nStat }}</h4>
    <h4 :data-tooltip="tooltip" @click="click">{{ value }}</h4>
    <div class="flexrow clickable" style="flex: 1; justify-content: center">
      <IronBtn icon="fa:subtract" @click="increment(-1)" />
      <IronBtn
        v-if="isMomentum"
        icon="fa:fire"
        @click="$actor?.burnMomentum()"
        :data-tooltip="burnMomentumTooltip"
      />
      <IronBtn icon="fa:plus" @click="increment(1)" />
    </div>
  </div>
</template>

<style lang="less" module>
@import (reference) '../../../styles/mixins.less';

.interactive {
  cursor: pointer;
  .interactiveMixin();

  &::before {
    --ironsworn-color-bg-highlight: var(--ironsworn-color-fg);

    transition: all 0.4s ease;
    opacity: 0;
    z-index: 0;
    padding: var(--ironsworn-spacer-sm);
  }

  &:hover {
    background-color: var(--ironsworn-color-clickable-block-bg-hover);

    &::before {
      background-color: ar(--ironsworn-color-fg);
      opacity: 1;
    }
  }
}

.wrapper {
  --ironsworn-color-text-stroke: var(--ironsworn-color-bg);
  .textStrokeMixin();

  h4 {
    font-weight: bold;

    margin: 0;
    padding: var(--ironsworn-spacer-sm) 0;
    text-transform: uppercase;
  }
}
</style>

<script setup lang="ts">
import { capitalize, computed, inject } from 'vue'
import { CharacterDataPropertiesData } from '../../actor/actortypes'
import { IronswornPrerollDialog } from '../../rolls'
import { $ActorKey, ActorKey } from '../provisions'
import IronBtn from './buttons/iron-btn.vue'

const { propKey } = defineProps<{
  propKey: string
}>()

const isMomentum = propKey === 'momentum'
const i18nKey = `IRONSWORN.${capitalize(propKey)}`
const i18nStat = game.i18n.localize(i18nKey)
const tooltip = computed(() =>
  isMomentum
    ? undefined
    : game.i18n.format('IRONSWORN.Roll +x', { stat: i18nStat })
)

const actor = inject(ActorKey)
const actorSystem = computed(
  () => (actor?.value as any)?.system as CharacterDataPropertiesData
)
const value = computed(() => actorSystem?.value?.[propKey])
const $actor = inject($ActorKey)

function increment(delta: number) {
  $actor?.update({ system: { [propKey]: value.value + delta } })
}

const burnMomentumTooltip = computed(() =>
  game.i18n.format('IRONSWORN.BurnMomentumAndResetTo', {
    value: actorSystem.value?.momentum,
    resetValue: actorSystem.value?.momentumReset,
  })
)

function click() {
  if (isMomentum) return
  IronswornPrerollDialog.showForStat(i18nStat, $actor?.system[propKey], $actor)
}
</script>
