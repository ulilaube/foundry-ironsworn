<template>
  <div class="flexcol" ref="root">
    <h1 class="flexrow">
      <span>{{ je().name }}</span>
      <IronBtn nogrow icon="ironsworn:d10-tilt" @click="randomize" />
    </h1>

    <TruthSelectable
      v-for="page in truthPages"
      ref="selectables"
      :page="page"
      :radio-group="df.$id"
      @change="valueChange"
    />

    <CustomTruth
      :radio-group="df.$id"
      ref="customTruth"
      @change="customValueChange"
    />

    <div
      class="nogrow"
      v-for="page in nonTruthPages"
      v-html="page.text.content"
    />
  </div>
</template>

<style lang="less" scoped>
h1 {
  margin-top: 1em;
}
</style>

<script lang="ts" setup>
import { ISettingTruth, ISettingTruthOption } from 'dataforged'
import { reactive, ref } from 'vue'
import { OracleRollMessage, TableRow } from '../../../rolls'
import { enrichMarkdown } from '../../vue-plugin'
import IronBtn from '../buttons/iron-btn.vue'
import CustomTruth from './custom-truth.vue'
import TruthSelectable from './truth-selectable.vue'

const props = defineProps<{
  df: ISettingTruth
  je: () => JournalEntry
}>()

const jePages = (props.je() as any | undefined)?.pages ?? []
const truthPages = jePages.filter((p) => p.type === 'truth')
const nonTruthPages = jePages.filter((p) => p.type !== 'truth')

const state = reactive<{
  title?: string
  text?: string
  html?: string
}>({})
function valueChange(title: string, text: string) {
  state.title = title
  state.text = text
  state.html = undefined
}
function customValueChange(html: string) {
  state.title = undefined
  state.text = undefined
  state.html = html
}

function selectedValue() {
  let html = state.html
  if (!html) {
    html = `
      ${enrichMarkdown(`**${state.title}**`)}
      ${enrichMarkdown(state.text)}
    `
  }
  html += nonTruthPages.map((x) => x.text.content).join('\n\n')

  return {
    title: props.je().name,
    html: html.trim(),
    valid: !!(state.title || state.html),
  }
}

const root = ref<HTMLElement>()
function scrollIntoView() {
  root.value?.scrollIntoView({
    behavior: 'smooth',
    block: 'start',
  })
}

const selectables = ref<typeof TruthSelectable[]>([])
const customTruth = ref<typeof CustomTruth>()

async function randomize() {
  // Roll it like an oracle
  const rows = truthPages
    .map((p) => p.system as ISettingTruthOption)
    .map(
      (sys: ISettingTruthOption): TableRow => ({
        low: sys.Floor || 0,
        high: sys.Ceiling || 100,
        text: sys.Result,
        selected: false,
      })
    )
  const msg = OracleRollMessage.fromRows(
    rows,
    props.je().name ?? '',
    game.i18n.localize('IRONSWORN.First Start.SettingTruths')
  )
  await msg.createOrUpdate()

  // Find the result and activate it
  const result = await msg.getResult()
  const dfRow = props.df.Table.find((x) => x.Floor === result?.low)
  if (!dfRow) throw new Error('wtf')
  const idx = props.df.Table.indexOf(dfRow)
  await selectables.value[idx]?.selectAndRandomize()
}

defineExpose({ selectedValue, scrollIntoView, randomize })
</script>
