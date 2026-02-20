<script setup lang="ts">
import { cn } from '@/lib/utils'
import { AccordionItem, type AccordionItemProps, useForwardProps } from 'reka-ui'
import { computed, type HTMLAttributes } from 'vue'

const props = defineProps<AccordionItemProps & { class?: HTMLAttributes['class'] }>()

const delegatedProps = computed(() => {
  const { class: _, ...delegated } = props

  return delegated
})

const forwardedProps = useForwardProps(delegatedProps)
</script>

<template>
  <AccordionItem 
    v-bind="forwardedProps" 
    :class="cn('rounded-[20px] overflow-hidden mb-2', props.class)"
    style="background: var(--bg-surface); box-shadow: 0 0 0 1px rgba(0,0,0,0.05);"
  >
    <slot />
  </AccordionItem>
</template>
