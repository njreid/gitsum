<script lang="ts">
  import Label from './Label.svelte'
  export let issue
</script>

<template lang="pug">
	tr
		td.tag
			b {issue.tags.category}
		td.tag
			b {issue.tags.platform}
		td.tag
			b {issue.tags.state}
		td.tag
			+each('issue.tags.other as label')
				Label(label="{label}")
		td
			a(href="{issue.html_url}" target="_blank") {issue.title}
		td
			a(href="{issue.user.html_url}" target="_blank") {issue.user.login}

</template>

<style lang="stylus">
	td
		padding 5px
		text-align left
		font-size 10pt

	.tag
		width 80px
</style>
