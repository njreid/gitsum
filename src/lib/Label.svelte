<script lang="ts">
  export let label


</script>

<template lang="pug">
	.label(style="background-color: #{label.color}")
		p {label.name}
</template>

<style lang="stylus">
	.label
		background-color blue
		color black
		padding 1px 10px
		font-size 7pt
		font-weight bold
		text-transform uppercase
		margin 2px
		border-radius 4px
		word-wrap none
		text-align center
</style>
