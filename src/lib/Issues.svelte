<script lang="ts">
  export let issues = []
  // import SvelteTable from 'svelte-table'
  // import * as punycode from 'punycode'
  // import md from 'markdown-it'
  import * as md from 'marked'
  import rake from 'rake-modified'

  import Words from './Words.svelte'

  let detail = []
  const DAY_MS = 1000 * 60 * 60 * 24

  import { onMount } from 'svelte'
  import samples from './sample.json'
  import { action_destroyer, identity } from 'svelte/internal'

  const platforms = {
    'react native': 'RN',
    typescript: 'TS',
    ios: 'iOS',
    android: 'And',
    angular: '<>',
    vue: 'Vue',
    react: 'React',
  }

  const states = {
    'pending-response': 'Respond',
    'pending-triage': 'Triage',
    investigating: 'Dig',
  }

  const issueTypes = {
    'feature-request': 'Feature',
    bug: 'Bug',
  }

  const categories = [
    'auth',
    'storage',
    'datastore',
    'graphql',
    'predictions',
    'notifications',
    'pubsub',
    'analytics',
    'interactions',
    'api',
    'appsync',
    'push notifications',
  ]

  let data = []
  let wordstore = []

  async function loadDetail(issueId) {
    const url = `https://api.github.com/repos/aws-amplify/amplify-js/issues/${issueId}`
    console.log(url)

    await fetch(url)
      .then(r => r.json())
      .then(data => {
        const html = md.parse(data.body)
        detail[issueId] = html
        const dom = new DOMParser()
        const doc = dom.parseFromString(html, 'text/html')
        let ps = Array.from(doc.getElementsByTagName('p'))
        let text = ps.reduce((a, c) => a + '\n' + c.innerText, '')
        const keywords = rake(text)
        const sortedKeywords = Object.entries(keywords).sort((a, b) => (a[1] < b[1] ? 1 : -1))
        wordstore[issueId] = sortedKeywords.slice(0,5)
        console.log(sortedKeywords)
        console.log(data)
      })
    // https://api.github.com/repos/aws-amplify/amplify-js/issues/9629
  }

  function indexIssues(issues) {
    let iss = []

    issues.map(issue => {
      let tags = {
        category: '',
        platform: '',
        state: '',
        type: '',
        other: [],
      }

      issue.labels.map(i => {
        let n = i.name.toLowerCase()
        console.log(n)
        let indexed = false

        if (n in platforms) {
          tags.platform = platforms[n]
          indexed = true
        }
        if (categories.includes(n)) {
          tags.category = n
          indexed = true
        }

        if (n in states) {
          tags.state = states[n]
          indexed = true
        }

        if (n in issueTypes) {
          tags.type = issueTypes[n]
          indexed = true
        }

        if (!indexed) {
          tags.other.push(i)
        }
      })

      issue.tags = tags
    })

    return issues
  }

  function ageInDays(d) {
    let ms = new Date().getTime() - new Date(d).getTime()
    return Math.floor(ms / DAY_MS)
  }

  onMount(async () => {
    // await fetch(`https://api.github.com/aws-amplify/amplify-js/issues`)
    //   .then(r => r.json())
    //   .then(data => {
    //     issues = data
    // 		console.log(data)
    //   })
    issues = indexIssues(samples)
    console.log(issues)
  })
</script>

<template lang="pug">
//- .detail {@html detail}
//- SvelteTable(columns="{columns}" rows="{issues}" showExpandIcon="{true}" expandRowKey="id")
.results
  table
    +each('issues as i')
      tr
        td.id {i.id}
        td {i.tags.type}
        td.category {i.tags.category}
        td {i.tags.platform}
        td.title(on:click="{loadDetail(i.number)}") {i.title}
      tr
        td(colspan="100%")
          Words(words="{wordstore[i.number] || []}")
      tr
        td(colspan="100%").detail {@html detail[i.number] || ""}

</template>

<!--
<style global>
  @import 'https://cdn.jsdelivr.net/npm/gridjs/dist/theme/mermaid.min.css';
</style> -->
<style lang="stylus">

table
  font-size 10pt
  border-spacing 0

td
  text-align left
  background-color #eee
  padding 4px 10px

.id
  font-size 8pt

.category
  text-transform uppercase
  font-weight bold
  font-size 9pt

.title
  font-weight bold



.words
  font-weight bold

.detail
  white-space pre
  text-align left
  font-size 9pt
  padding 4px
  background-color white
  color #888
  max-width 600px
  white-space normal
  margin 0
  padding-left 10px

  :global(img)
    max-width 500px

  :global(h3)
    padding 0
    margin-block-end 4px
  :global(p)
    margin 0

</style>
