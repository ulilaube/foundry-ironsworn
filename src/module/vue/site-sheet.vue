<template>
  <div class="flexcol" :class="$style.siteSheet">
    <!-- HEADER -->
    <SheetHeaderBasic class="nogrow" :document="data.actor" />
    <div class="flexrow nogrow" :class="$style.main">
      <div class="flexcol" :class="$style.leftColumn">
        <!-- RANK -->
        <article :class="$style.progressWidget">
          <div class="flexrow nogrow" :class="$style.rankRow">
            <RankPips
              :current="data.actor.system.rank"
              class="nogrow"
              @click="setRank"
              :id="`${data.actor._id}_rank`"
            />
            <label :for="`${data.actor._id}_rank`" :class="$style.rankLabel">{{
              rankText
            }}</label>
            <IronBtn
              v-if="editMode"
              block
              nogrow
              @click="clearProgress"
              icon="fa:trash"
            />
            <IronBtn block nogrow @click="markProgress" icon="fa:caret-right" />
          </div>
          <!-- PROGRESS -->
          <ProgressTrack
            class="nogrow"
            :ticks="data.actor.system.current"
            :rank="data.actor.system.rank"
          />
        </article>
        <!-- THEME/DOMAIN -->
        <div class="boxgroup flexcol nogrow">
          <div class="flexrow boxrow nogrow">
            <SiteDroparea
              class="box"
              :item="theme"
              item-type="delve-theme"
              compendium-key="ironsworndelvethemes"
              title-key="IRONSWORN.ITEM.TypeDelveTheme"
            />
          </div>
          <div class="flexrow boxrow nogrow">
            <SiteDroparea
              class="box"
              :item="domain"
              item-type="delve-domain"
              compendium-key="ironsworndelvedomains"
              title-key="IRONSWORN.ITEM.TypeDelveDomain"
            />
          </div>
        </div>
        <!-- DENIZENS -->
        <article :class="$style.denizenMatrix">
          <h2 class="flexrow nogrow" :class="$style.heading">
            <span>{{ $t('IRONSWORN.DELVESITE.Denizens') }}</span>
            <IronBtn
              nogrow
              style="padding: 2px"
              @click="randomDenizen"
              icon="ironsworn:d10-tilt"
            />
            <BtnCompendium compendium="ironswornfoes" nogrow />
          </h2>
          <div class="boxgroup nogrow">
            <div class="flexrow boxrow">
              <SiteDenizenbox :idx="0" :ref="(e) => (denizenRefs[0] = e)" />
              <SiteDenizenbox :idx="1" :ref="(e) => (denizenRefs[1] = e)" />
            </div>
            <div class="flexrow boxrow">
              <SiteDenizenbox :idx="2" :ref="(e) => (denizenRefs[2] = e)" />
              <SiteDenizenbox :idx="3" :ref="(e) => (denizenRefs[3] = e)" />
            </div>
            <div class="flexrow boxrow">
              <SiteDenizenbox :idx="4" :ref="(e) => (denizenRefs[4] = e)" />
              <SiteDenizenbox :idx="5" :ref="(e) => (denizenRefs[5] = e)" />
            </div>
            <div class="flexrow boxrow">
              <SiteDenizenbox :idx="6" :ref="(e) => (denizenRefs[6] = e)" />
              <SiteDenizenbox :idx="7" :ref="(e) => (denizenRefs[7] = e)" />
            </div>
            <div class="flexrow boxrow">
              <SiteDenizenbox :idx="8" :ref="(e) => (denizenRefs[8] = e)" />
              <SiteDenizenbox :idx="9" :ref="(e) => (denizenRefs[9] = e)" />
            </div>
            <div class="flexrow boxrow">
              <SiteDenizenbox :idx="10" :ref="(e) => (denizenRefs[10] = e)" />
              <SiteDenizenbox :idx="11" :ref="(e) => (denizenRefs[11] = e)" />
            </div>
          </div>
        </article>
      </div>
      <div class="scrollable flexcol" :class="$style.rightColumn">
        <SiteMoves class="nogrow" />
      </div>
    </div>
    <div class="flexcol">
      <div class="flexrow nogrow">
        <h2 :class="$style.heading">{{ $t('Notes') }}</h2>
        <IronBtn
          block
          nogrow
          class="box"
          :disabled="!hasThemeAndDomain"
          :class="{ [$style.featureBtn]: true }"
          @click="randomFeature"
          icon="ironsworn:d10-tilt"
          :text="$t('IRONSWORN.DELVESITE.Feature')"
        />
      </div>
      <MceEditor
        v-model="data.actor.system.description"
        @save="saveDescription"
      />
    </div>
  </div>
</template>

<style lang="less" module>
.siteSheet {
  gap: 0.5em;
}

.rankRow {
  gap: var(--ironsworn-spacer-lg);
}

.rankLabel {
  display: flex;
  flex-direction: row nowrap;
  align-items: center;
  margin: 0;
  text-transform: uppercase;
  line-height: 22px;
  font-size: var(--font-size-14);
}

.denizenMatrix {
  // TODO: extract this as its own component
}

.main {
  gap: inherit;
}

.siteMoves {
  height: max-content;
}

.rightColumn {
  flex-basis: 12em;
  max-height: 411px;
}

.leftColumn {
  flex-basis: 20em;
  gap: 1em;
}

.heading {
  display: flex;
  align-items: center;
  margin: 0;
  text-transform: uppercase;
  line-height: 1.5;
  font-size: var(--font-size-14);
  font-weight: bold;
}

.featureBtn {
  text-transform: uppercase;
}
</style>

<style lang="less" scoped>
textarea {
  border-radius: var(--ironsworn-border-radius-sm);
  resize: none;
}
</style>

<script setup lang="ts">
import SheetHeaderBasic from './sheet-header-basic.vue'
import { provide, computed, inject, nextTick, ref } from 'vue'
import { $ActorKey, ActorKey } from './provisions'
import RankPips from './components/rank-pips/rank-pips.vue'
import BtnCompendium from './components/buttons/btn-compendium.vue'
import SiteDroparea from './components/site/site-droparea.vue'
import SiteDenizenbox from './components/site/site-denizenbox.vue'
import MceEditor from './components/mce-editor.vue'
import { RANKS, RANK_INCREMENTS } from '../constants'
import { createIronswornDenizenChat } from '../chat/chatrollhelpers'
import ProgressTrack from './components/progress/progress-track.vue'
import SiteMoves from './components/site/site-moves.vue'
import { OracleRollMessage } from '../rolls'
import { DelveThemeDataSourceData } from '../item/itemtypes'
import IronBtn from './components/buttons/iron-btn.vue'
import { SiteDataPropertiesData } from '../actor/actortypes'

const props = defineProps<{
  data: { actor: any }
}>()

provide(ActorKey, computed(() => props.data.actor) as any)
provide('toolset', 'ironsworn')

const $actor = inject($ActorKey)

const editMode = computed(() => {
  return (props.data.actor.flags['foundry-ironsworn'] as any)?.['edit-mode']
})

const theme = computed(() => {
  return props.data.actor.items.find((x) => x.type === 'delve-theme')
})

const domain = computed(() => {
  return props.data.actor.items.find((x) => x.type === 'delve-domain')
})

const rankText = computed(() => {
  return game.i18n.localize(RANKS[props.data.actor.system.rank])
})

function setRank(rank) {
  $actor?.update({ system: { rank } })
}

function clearProgress() {
  $actor?.update({ 'system.current': 0 })
}

function markProgress() {
  const increment = RANK_INCREMENTS[props.data.actor.system.rank]
  const newValue = Math.min(props.data.actor.system.current + increment, 40)
  $actor?.update({ 'system.current': newValue })
}

const denizenRefs = ref<{ [k: number]: any }>({})
async function randomDenizen() {
  const roll = await new Roll('1d100').evaluate({ async: true })
  const result = roll.total
  const denizens = (props.data.actor.system as SiteDataPropertiesData).denizens
  const denizen = denizens.find(
    (x) => x.range[0] <= result && x.range[1] >= result
  )
  if (!denizen) throw new Error(`Rolled a ${result} but got no denizen???`)
  const idx = props.data.actor.system.denizens.indexOf(denizen)
  await createIronswornDenizenChat({
    roll,
    denizen,
    site: $actor!,
  })

  // Denizen slot is empty; set focus and add a class
  if (!denizen?.text) {
    await $actor?.setFlag('foundry-ironsworn', 'edit-mode', true)
    await nextTick()
    denizenRefs.value[idx]?.focus?.()
  }
}

const hasThemeAndDomain = computed(() => {
  return !!(theme.value && domain.value)
})

async function randomFeature() {
  if (!hasThemeAndDomain.value) return

  const themeData = (theme.value as any)?.system as DelveThemeDataSourceData
  const domainData = (domain.value as any)?.system as DelveThemeDataSourceData
  const title = game.i18n.localize('IRONSWORN.DELVESITE.Feature')
  const subtitle = `${$actor?.name} – ${theme.value?.name} ${domain.value?.name}`
  const orm = OracleRollMessage.fromTableResults(
    [...themeData.features, ...domainData.features],
    title,
    subtitle
  )
  orm.createOrUpdate()
}

function saveDescription() {
  $actor?.update({ 'system.description': props.data.actor.system.description })
}
</script>
