<template>
  <div class="company-head columns">
    <div class="column is-1">
      <FallbackIcon
        v-if="company.name"
        :chars="company.name"
        :img="'https://logo.clearbit.com/'+company.url"
      />
      <!-- <img
        class="company-head__img"
        :src="company.logo || 'https://logo.clearbit.com/'+company.url"
      />-->
    </div>
    <div class="column is-8">
      <div class="company-head__name">{{company.name}}</div>
      <!-- <div class="company-head__domain">Technology company</div> -->
      <!-- <div class="company-head__description">
        Microsoft Corporation is an American multinational technology company with headquarters in Redmond, Washington. It develops, manufactures, licenses, supports and sells computer software, consumer electronics, personal computers, and related services.</div>
      -->
    </div>
    <div class="column is-3">
      <div class="company-head__info-wrapper">
        <a class="company-head__info" target="_blank" :href="'http://'+company.url">
          <span class="icon is-small is-left">
            <i class="material-icons md-24">link</i>
          </span>
          <span class="company-head__info-val">{{company.url}}</span>
        </a>
        <!-- <a class="company-head__info" target="_blank" href="https://fb.com/Microsoft">
          <span class="icon is-small is-left">
                <font-awesome-icon :icon="{ prefix: 'fab', iconName: 'facebook' }" />
          </span>
          <span class="company-head__info-val">MicrosoftWeb</span>
        </a>
        <a class="company-head__info" target="_blank" href="https://twitter.com/Microsoft">
          <span class="icon is-small is-left">
                <font-awesome-icon :icon="{ prefix: 'fab', iconName: 'twitter' }" />
          </span>
          <span class="company-head__info-val">Microsoft</span>
        </a>-->
      </div>
    </div>
  </div>
</template>
<script>
import FallbackIcon from "~/components/FallbackIcon.vue";
export default {
  name: "CompanyHeader",
  props: ["company"],
  components: {
    FallbackIcon
  }
};
</script>
<style lang="sass">
$color-primary: #2196f3
$color-white: #fff
$color-text-1: #111
$color-text-2: #444
$color-text-3: #666
$box-shadow: 0 1px 6px 1px rgba(92,78,62,.28)

$font-size-2xl: 22px
$font-size-s: 14px

.company
    &-head
        border-bottom: 1px solid #dfe1e8
        border-radius: 0
        margin: 10px 0
        padding: 10px
        color: $color-text-1
        border-radius: 4px
        display: flex
        align-items: center
        &__img
            max-height: 48px
            max-width: 48px
            display: block
        &__name
            font-size: $font-size-2xl
            line-height: 1
            font-weight: 500
        &__domain
            color: $color-text-3
            font-size: $font-size-s
            margin-top: 5px
            font-weight: 300

        &__description
            margin-top: 10px
            color: $color-text-2
        
        &__info
            list-style-type: none
            display: block
            &-wrapper
                color: $color-text-2
                margin-left: 40px
            .icon
                vertical-align: middle
            &-val
                vertical-align: middle
                margin-left: 5px
            & + &
                margin-top: 5px
</style>
