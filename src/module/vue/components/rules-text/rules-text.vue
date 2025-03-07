<template>
  <WithRolllisteners element="article" class="rules-text">
    <slot name="before-main"></slot>
    <section v-if="props.type === 'slot'" class="rules-text-main">
      <slot name="default"></slot>
    </section>
    <section v-else class="rules-text-main" v-html="enrichedText" />
    <slot name="after-main"></slot>
    <footer class="rules-text-footer" v-if="props.source">
      <RulesSourceInfo :source="props.source" />
    </footer>
    <slot name="after-footer"></slot>
  </WithRolllisteners>
</template>
<style lang="less">
@import (reference) '../../../../styles/mixins.less';

.rules-text {
  --ironsworn-rules-text-spacer: 0.5em;

  display: flex;
  flex-direction: column;
  gap: var(--ironsworn-rules-text-spacer);
}

.rules-text-main {
  display: flex;
  flex-direction: column;
  gap: var(--ironsworn-rules-text-spacer);

  p {
    &:first-of-type {
      margin-top: 0;
    }

    &:last-of-type:not(:last-child) {
      margin-bottom: 0;
    }
  }

  & > * {
    margin: 0;
  }
  .textCompactMixin();
}

.rules-text-footer {
  text-align: right;
}

.rules-source-info {
  opacity: 0.5;
  font-size: small;
}
</style>

<script setup lang="ts">
import { computed } from 'vue'
import { IronswornHandlebarsHelpers } from '../../../helpers/handlebars.js'
import { enrichMarkdown } from '../../vue-plugin.js'
import RulesSourceInfo from './rules-source-info.vue'
import WithRolllisteners from '../with-rolllisteners.vue'
import { ISource } from 'dataforged'
import { enrichHtml } from '../../vue-plugin.js'

const props = defineProps<{
  /**
   * The type of text content to be displayed:
   * `slot`: uses the content of the 'default' slot
   * `markdown`: the `content` prop, and passes it through {@link enrichMarkdown}
   * `html`: the `content` prop, and passes it through {@link enrichHtml}
   */
  type: 'slot' | 'markdown' | 'html'
  /**
   * Markdown or HTML content, used when `type` is set to `markdown` or `html`
   */
  content?: string
  /**
   * Dataforged source data to be included in the footer.
   */
  source?: ISource
  /**
   * Whether or not to strip tables from the display. Many times they are redundant,
   * but sometimes they are essential.
   */
  stripTables?: boolean
}>()

const enrichedText = computed(() => {
  if (props.type === 'markdown') {
    const html = enrichMarkdown(props.content as string)
    if (props.stripTables) return IronswornHandlebarsHelpers.stripTables(html)
    return html
  }
  if (props.type === 'html') {
    return IronswornHandlebarsHelpers.stripTables(enrichHtml(props.content))
  }
  return ''
})
</script>
