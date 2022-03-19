<script lang="ts">
  export let words = []
</script>

<template lang="pug">

.words
  +each('words as word')
    .word {word[0]}

</template>

<style lang="stylus">

  .words
    display flex

  .word
    padding 6px
    background-color #071252
    color white
    width 100px
    border-radius 4px
    margin-right 2px

</style>
