<script lang="ts">
  export let issues = []
  import Issue from './Issue.svelte'
  import { onMount } from 'svelte'

  import samples from './sample.json'

  const platforms = ['react native', 'typescript', 'ios', 'android', 'angular', 'vue']
  const states = ['pending-response', 'pending-triage', 'investigating']
  const categories = ['auth', 'storage', 'datastore', 'graphql', 'predictions', 'notifications', 'pubsub', 'analytics', 'interactions', 'api', 'appsync', 'push notifications']

  function indexIssues(issues) {
    issues.map(issue => {
      let tags = {
        category: "",
        platform: "",
        state: "",
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
    })
    return issues
  }

  function sortBy(field) {
    return () => {
      console.log('Sorting by ' + field)
      issues.sort((a, b) => {
        if (a.tags[field] > b.tags[field]) return 1
        if (a.tags[field] < b.tags[field]) return -1
      })
			issues = issues
    }
  }

  onMount(async () => {
    // await fetch(`https://api.github.com/aws-amplify/amplify-js/issues`)
    //   .then(r => r.json())
    //   .then(data => {
    //     issues = data
    // 		console.log(data)
    //   })
    issues = indexIssues(samples)
  })
</script>

<template lang="pug">
h1 Issues list
table
	tr
		td(on:click="{sortBy('categories')}")
			p Category
		td(on:click="{sortBy('platforms')}") Platform
		td(on:click="{sortBy('states')}") States
		td(on:click="{sortBy('other')}") Other
		td Title
		td User
	+each('issues as issue')
		Issue(issue="{issue}")
</template>


<style lang='stylus'>

	td
		padding 4px
		cursor pointer

</style>