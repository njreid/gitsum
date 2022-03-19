<script lang="ts">
  export let issues = []
  import SvelteTable from 'svelte-table'

  let detail = ''
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

  async function loadDetail(issueId) {
    await fetch(`https://api.github.com/aws-amplify/amplify-js/issues/${issueId}`)
      .then(r => r.json())
      .then(data => {
        detail = data.body
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

  const columns = [
    {
      key: 'id',
      title: 'ID',
      value: v => v.id,
      sortable: true,
      class: 'id',
      headerClass: 'text-left',
    },
    {
      key: 'type',
      title: 'T',
      value: v => v.tags.type,
      sortable: true,
      headerClass: 'text-left',
    },
    {
      key: 'category',
      title: 'Category',
      value: v => v.tags.category.toUpperCase(),
      sortable: true,
      class: 'category',
      filterOptions: rows =>
        categories.reduce((a, c) => {
          a.push({ name: c, value: c.toUpperCase() })
          return a
        }, []),
      headerClass: 'text-left',
    },
    {
      key: 'platform',
      title: 'Platform',
      value: v => v.tags.platform.toUpperCase(),
      sortable: true,
      class: 'platform',
      filterOptions: rows =>
        Object.entries(platforms).reduce((a, c) => {
          a.push({ name: c[1], value: c[1] })
          return a
        }, []),
      headerClass: 'text-left',
    },
    {
      key: 'state',
      title: 'State',
      value: v => v.tags.state,
      sortable: true,
      headerClass: 'text-left',
    },

    {
      key: 'title',
      title: 'Title',
      value: v => `<a on:click="{loadDetail(v.number)}">${v.title}</a>`,
      // value: v => `<a href='${v.html_url}' target='_blank'>${v.title}</a>`,
      sortable: true,
      headerClass: 'title',
      class: 'title',
    },
    {
      key: 'other',
      title: 'Tags',
      value: v => v.tags.other?.reduce((a, c) => a + c.name, '') || '',
      sortable: true,
      headerClass: 'text-left',
    },
    {
      key: 'created',
      title: 'Age',
      value: v => ageInDays(v.created_at),
      sortable: true,
      headerClass: 'text-left',
    },
    {
      key: 'updated',
      title: 'Last',
      value: v => ageInDays(v.updated_at),
      sortable: true,
      headerClass: 'text-left',
    },
    {
      key: 'comments',
      title: 'Comments',
      value: v => v.comments,
      sortable: true,
      headerClass: 'text-left',
    },
  ]
</script>

<template lang="pug">
.detail {detail}
SvelteTable(columns="{columns}" rows="{issues}" showExpandIcon="{true}" expandRowKey="id")

</template>

<!--
<style global>
  @import 'https://cdn.jsdelivr.net/npm/gridjs/dist/theme/mermaid.min.css';
</style> -->
<style lang="stylus">

:global(table)
  font-size 10pt
  // max-width 90%

:global(thead)
  background-color #222
  color white
  :global(th)
    padding 6px
  :global(td)
    padding 2px

:global(td.left)
  text-align left

:global(.title)
  max-width 500px
  text-align left
  white-space normal

:global(td)
  padding 2px
  white-space nowrap
  overflow hidden

:global(.platform)
  max-width 80px
  font-size 8pt
  font-weight 800

:global(.category)
  max-width 80px
  font-size 8pt
  font-weight 800
  background-color #eee

:global(.id)
  font-size 8pt

</style>
