<template>
  <div class="row topic-page">
    <div class="col-lg-8 col-md-8 col-sm-12">
      <slot name="content"/>
    </div>
    <div class="col-lg-4 col-md-4 col-sm-12">
      <slot name="side"/>
    </div>
  </div>
</template>

<style scoped lang="stylus">
  .topic-page
    margin-top 5px

    .topicpage-header
      display flex
      align-items center
      justify-content space-between

      .header-title
        margin 15px 0
        font-weight 800
        font-size 2.7rem
        display flex
        align-items center

    .topic-logo
      margin-top 10px
      margin-bottom 30px

    .topic-logo-edit
      border 1px solid #e9ecef

    .content-block
      >>> img
        max-width 95%
        margin 20px auto
        display block
</style>
