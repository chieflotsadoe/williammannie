<template>
  <nav class="nav">
    <h1 class="nav__title">
      <router-link to="/blog">{{ content.title }}</router-link>
    </h1>

    <transition-group tag="menu" name="nav__item" class="nav__menu">
      <li v-for="label in labels" class="nav__item" :key="label" @click="navBack">
        <i class="nav__item--icon"></i>
        <span class="nav__item--label">{{ label }}</span>
      </li>
    </transition-group>
  </nav>
</template>

<script>
export default {
  name: 'blog-nav',
  props: {
    navs: Number,
    content: Object,
    filters: {
      type: Object,
      default: () => {}
    }
  },

  computed: {
    labels () {
      return Object.keys(this.filters)
        .map(filter => this.content.labels[filter])
    }
  },

  methods: {
    navBack () {
      if (this.navs && !this.filters.author) this.$router.go(-1)
      else this.$router.push('/')
    }
  }
}
</script>

<style>
  .nav {
    position: fixed;
    top: 0;
    left: 0;
    z-index: 10;
    height: 50px;
    width: 100%;
    background-color: black;
  }
  .nav h1 {
    font-size: 10px;
  }

</style>
