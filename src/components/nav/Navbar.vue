<template>
  <nav class="Navbar">
    <Logo
      v-if="showLogo !== false"
      :class="`Navbar__Logo Navbar__Logo--display-${showLogo}`"
    />
    <div
      v-dragscroll.x="true"
      class="main-itens"
    >
      <nuxt-link
        v-for="(page, index) in pageLinks()"
        :key="index"
        class="Navbar__Item"
        :to="$utils.urlOrRoute(page)"
      >
        {{ page.title }}
      </nuxt-link>
    </div>
    <NavbarControls />
  </nav>
</template>

<script>
import Logo from '@/components/nav/Logo'
import NavbarControls from '@/components/nav/NavbarControls'
import { mapGetters } from 'vuex'
import ClickOutside from 'vue-click-outside'

export default {
  name: 'Navbar',
  components: {
    Logo,
    NavbarControls
  },
  directives: {
    ClickOutside
  },
  props: {
    showLogo: {
      type: [String, Boolean],
      default: 'auto',
      validator (value) {
        return [
          true, // exiba o logotipo sempre
          false, // esconda o logotipo sempre
          'auto', // mostre o logotipo se houver espaço
          'mobile', // sempre mostre o logo apenas em mobile
          'desktop' // sempre mostre o logo em desktop
        ].includes(value)
      }
    }
  },
  computed: mapGetters(['pageLinks'])
}
</script>

<style lang="stylus" scoped>
.Navbar
  font-family: $menu-font
  font-size: $text-size
  z-index: 900
  max-width: $max-width
  margin: 0 auto
  width: 100%
  position: relative
  justify-content: space-between
  align-items: stretch
  display: flex
  min-height: 80px
  padding: 0 0.5rem
  @media only screen and (min-width: $tablet)
    min-height: 100px

.Navbar__Logo
.main-itens
.controls
.main-itens a
.controls a
  align-items: center
  display: flex

.Navbar__Logo
  transition: opacity 0.25s ease

.main-itens
.controls
  align-items: stretch

.main-itens a
.controls a
  color: #555
  margin: 0 0.5rem
  outline: none
  transition: color 0.25s ease

nav a
  &.nuxt-link-exact-active
  &:hover
  &:active
  &:focus
    color: $link-color-active

.main-itens
  display: none
  @media only screen and (min-width: $tablet)
    display: flex

</style>
