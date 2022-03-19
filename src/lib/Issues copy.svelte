<script lang="ts">
  export let issues = []

  import { RevoGrid } from '@revolist/svelte-datagrid'
  import { defineCustomElements } from '@revolist/revogrid/loader'

  let source
  let headers

  // define webcomponent element in DOM
  defineCustomElements().then(() => {
    // then apply data or rerender
    source = [
      {
        prop: 'id',
        name: 'Id',
      },
      {
        prop: 'title',
        name: 'Title',
      },
    ]
    headers = [
      {
        name: '1',
        details: 'Item 1',
      },
    ]
  })

  function onEdit(e) {
    console.log(e)
  }

  import { onMount } from 'svelte'
  import samples from './sample.json'

  const platforms = ['react native', 'typescript', 'ios', 'android', 'angular', 'vue']
  const states = ['pending-response', 'pending-triage', 'investigating']
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
  let grid

  function indexIssues(issues) {
    let iss = []

    issues.map(issue => {
      let tags = {
        category: '',
        platform: '',
        state: '',
        other: [],
      }

      issue.labels.map(i => {
        let n = i.name.toLowerCase()
        console.log(n)
        let indexed = false

        if (platforms.includes(n)) {
          tags.platform = n
          indexed = true
        }
        if (categories.includes(n)) {
          tags.category = n
          indexed = true
        }

        if (states.includes(n)) {
          tags.state = n
          indexed = true
        }

        if (!indexed) {
          tags.other.push(i)
        }
      })

      console.log(tags)
      issue.tags = tags
      iss.push({
        id: issue.id,
        title: issue.title,
        category: issue.tags.category,
        platform: issue.tags.platform,
      })

      // iss.push([
      //    issue.id,
      //    issue.tags.category,
      //    issue.tags.platform,
      //    issue.tags.state,
      //    issue.title,
      // ])
    })

    return iss
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
    console.log(grid)
    // grid.updateConfig({
    //   columns: ['Id', 'Category', 'Platform', 'State', 'Title'],
    //   data: issues,
    // })
    // grid.data = issues
  })
</script>

<template lang="pug">
h1 Issues list
RevoGrid(class="grid" on:beforeEdit="{onEdit}" source="{source}" resize="true" columns="{headers}" theme="material")
</template>
<!--
<style global>
  @import 'https://cdn.jsdelivr.net/npm/gridjs/dist/theme/mermaid.min.css';
</style> -->
