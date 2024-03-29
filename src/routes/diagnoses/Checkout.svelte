<script lang="ts">
  import { getToastStore } from "@skeletonlabs/skeleton"
  import type { DecisionTree } from "$lib/utils"
  import { postMarkdownToDocx } from "$lib/api"

  export let nodes: DecisionTree[]

  let templates = getTemplateText(nodes)
  let userTemplates = templates.map(template => allUpperCaseUnderscoreToCapitalizedSpace(template))
  let values = Array(templates.length).fill("")

  const toastStore = getToastStore()

  function onSubmit(event: Event) {
    event.preventDefault()
    if (values.some(value => value === "")) {
      toastStore.trigger({ message: "Please fill all the fields.", background: "variant-filled-error" })
      return
    }

    let text = nodes.map(node => node.text).join("\n\n")
    templates.forEach((template, index) => {
      text = text.replace(new RegExp(`\{\{${template}\}\}`, "g"), values[index])
    })
    postMarkdownToDocx(text)
  }

  function allUpperCaseUnderscoreToCapitalizedSpace(input: string): string {
    return input
      .split("_")
      .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
      .join(" ")
  }

  function getTemplateText(nodes: DecisionTree[]): string[] {
    const templates = new Set<string>()
    nodes.forEach(node => {
      const matches = node.text.match(/{{(.*?)}}/g)
      if (!matches) return
      matches.forEach(match => {
        const template = match.replace(/{{|}}/g, "").trim()
        templates.add(template)
      })
    })
    return Array.from(templates)
  }
</script>

<div class="space-y-2">
  {#if userTemplates.length === 0}
    <p class="text-center">No fields required.</p>
  {:else}
    <p class="text-center">Please fill in the following fields:</p>
  {/if}
  <div class="space-x-2">
    {#each userTemplates as template, index}
      <input class="input max-w-60" type="text" placeholder={template} bind:value={values[index]} />
    {/each}
  </div>
  <button class="btn variant-filled-primary" on:click={onSubmit}>Download</button>
</div>
