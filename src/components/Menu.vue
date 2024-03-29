<template>
  <div :class="['v-menu', 'clearfix', classes]">
    <!-- Wrap token to fix Desktop Safari 5 -->
    <!-- https://developers.google.com/web/tools/lighthouse/audits/passive-event-listeners -->
    <div
      ref="token"
      @click="toggle"
      :style="inlineTokenStyle"
      :class="['token', { 'is-open': classes['is-open'] }]"
    ></div>
    <ul
      ref="menu"
      :style="inlineMenuStyle"
      :class="['menu', menuClass, { 'is-open': classes['is-open'] }]"
    >
      <li>
        <slot name="before"></slot>
      </li>
      <template v-for="(route, index) in flatRoutes">
        <li
          :key="index"
          :class="{
            'is-selected': route.path === currRoute.path,
            'is-child-item': route.nested && route.path !== route.parent.path,
          }"
          v-if="route.name"
        >
          <router-link :to="{ name: route.name }">
            {{ route.name }}
          </router-link>
        </li>
      </template>
      <slot v-if="false"></slot>
      <!-- https://forum.vuejs.org/t/loop-with-v-for-slots-default/20646/7 -->
      <v-wrap-node
        tag="li"
        :value="node"
        :key="`node-${index}`"
        v-for="(node, index) of $slots.default"
      ></v-wrap-node>
    </ul>
  </div>
</template>

<script>
// https://github.com/vuejs/vue-router/issues/800
// https://developer.mozilla.org/en-US/docs/Web/Events/mouseout
// https://developer.mozilla.org/en-US/docs/Web/Events/mouseleave
// https://gist.github.com/languanghao/5f74ca361f22192ba774941a69fd275b
// https://developers.google.com/web/updates/2016/06/passive-event-listeners
// https://stackoverflow.com/questions/4616694/what-is-event-bubbling-and-capturing
// https://stackoverflow.com/questions/39589911/svg-image-not-working-in-safari-5-1-7-windows

import { reject } from 'lodash'
import VWrapNode from '@/components/WrapNode'
import flattenRoutes from '@/vue-flatten-routes'
// import { flattenRoutes } from 'vue-flatten-routes'

export default {
  name: 'VMenu',
  props: {
    routes: {
      type: Array,
      default: () => [],
    },
    exclude: {
      type: Array,
      default: () => [],
    },
    menuClass: {
      type: String,
    },
    tokenStyle: {
      type: Object,
      default: () => ({}),
    },
    menuStyle: {
      type: Object,
      default: () => ({}),
    },
  },
  components: {
    VWrapNode,
  },
  data: () => ({
    open: false,
    currRoute: null,
  }),
  // https://stackoverflow.com/questions/46402809/vuejs-event-on-route-change
  watch: {
    $route: {
      handler(to) {
        this.currRoute = to
      },
      deep: true,
      immediate: true,
    },
  },
  mounted() {
    document.addEventListener('click', this.dismiss)
    window.addEventListener('resize', this.handleResize)
  },
  beforeDestroy() {
    document.removeEventListener('click', this.dismiss)
    window.removeEventListener('resize', this.handleResize)
  },
  computed: {
    classes() {
      return {
        'is-open': this.open,
      }
    },
    flatRoutes() {
      return reject(flattenRoutes(this.routes), route =>
        this.exclude.includes(route.name || route.path)
      )
    },
    inlineTokenStyle() {
      return {
        // <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 196.32 170.02"><defs><style>.cls-1{fill:#42b883;}.cls-2{fill:#35495e;}</style></defs><title>logo</title><polygon class="cls-1" points="120.83 0 98.16 39.26 75.49 0 0 0 98.16 170.02 196.32 0 120.83 0"/><polygon class="cls-2" points="120.83 0 98.16 39.26 75.49 0 39.26 0 98.16 102.01 157.06 0 120.83 0"/></svg>
        backgroundImage: `url(data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxOTYuMzIgMTcwLjAyIj48ZGVmcz48c3R5bGU+LmNscy0xe2ZpbGw6IzQyYjg4Mzt9LmNscy0ye2ZpbGw6IzM1NDk1ZTt9PC9zdHlsZT48L2RlZnM+PHRpdGxlPmxvZ288L3RpdGxlPjxwb2x5Z29uIGNsYXNzPSJjbHMtMSIgcG9pbnRzPSIxMjAuODMgMCA5OC4xNiAzOS4yNiA3NS40OSAwIDAgMCA5OC4xNiAxNzAuMDIgMTk2LjMyIDAgMTIwLjgzIDAiLz48cG9seWdvbiBjbGFzcz0iY2xzLTIiIHBvaW50cz0iMTIwLjgzIDAgOTguMTYgMzkuMjYgNzUuNDkgMCAzOS4yNiAwIDk4LjE2IDEwMi4wMSAxNTcuMDYgMCAxMjAuODMgMCIvPjwvc3ZnPg==)`,
        ...this.tokenStyle,
      }
    },
    inlineMenuStyle() {
      return this.menuStyle
    },
  },
  methods: {
    toggle(state) {
      this.open = typeof state === 'boolean' ? state : !this.open
      this.$emit('toggle', this.open)
      this.emitState()
    },
    dismiss(event) {
      if (event.target === this.$refs.token) return
      if (this.open) this.toggle(false)
    },
    async emitState(state = this.open) {
      const width = await this.getWidth()
      this.$emit(state ? 'open' : 'close', { width })
    },
    handleResize() {
      this.emitState(this.open)
    },
    getWidth() {
      return new Promise(resolve => {
        window.requestAnimationFrame(() => {
          resolve(this.$refs.menu.getBoundingClientRect().width)
        })
      })
    },
  },
}
</script>

<style lang="scss" scoped>
@import '~bootstrap/scss/functions';
@import '~bootstrap/scss/variables';
@import '~bootstrap/scss/mixins';

.v-menu {
  left: 1rem;
  bottom: 0.5rem;
  display: flex;
  direction: ltr;
  z-index: 999999;
  flex-direction: column;
  justify-content: flex-end;
}

.token {
  left: 2rem;
  width: 4rem;
  height: 4rem;
  bottom: 3rem;
  opacity: 0.5;
  display: flex;
  flex-shrink: 0;
  padding: 0.5rem;
  z-index: 999999;
  position: fixed;
  user-select: none; // Fix Firefox
  justify-content: center;
  background-position: center;
  background-repeat: no-repeat;
  background-size: 1.2rem 1.2rem;
  transition: opacity var(--transition-duration),
    transform var(--transition-duration) ease;
  transform: translate(-1.4rem, 1.4rem);

  &:hover {
    opacity: 1;
  }

  @include media-breakpoint-down(xs) {
    width: 3rem;
    height: 3rem;
    bottom: 2rem;
  }

  &.is-open {
    left: auto;
    right: 2rem;
    opacity: 1;
    position: fixed;
    transform: translate(1.4rem, 1.4rem);
  }
}

.menu {
  top: 0;
  left: 0;
  bottom: 0;
  margin: 0;
  width: 50vw;
  z-index: 999999;
  min-width: 160px;
  flex-grow: 1;
  display: flex;
  overflow: auto;
  position: fixed;
  overflow-x: hidden; // Fix IE
  flex-direction: column;
  transform: translateX(-100%);
  transition: transform 0.3s ease;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);

  &.is-open {
    transform: translateX(0);
  }
}

.is-child-item {
  padding-left: 1rem;
}

.is-selected {
  &::before {
    display: block;
    content: '';
    height: 1rem;
    width: 0.01rem;
    margin-right: 1rem;
    background-color: #8b9dc3;
  }
}

.clearfix {
  &::after,
  &::before {
    height: 0;
    display: block;
    content: '\0020';
    overflow: hidden;
  }

  &::after {
    clear: both;
  }
}

ul {
  margin: 0;
  padding: 0;
  list-style-type: none;
}

li:not(:empty) {
  display: flex;
  align-items: center;
  min-height: 4rem;
  margin-left: 1rem;
  margin-right: 1rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 0.01rem solid rgba(255, 255, 255, 0.05);

  @include media-breakpoint-down(xs) {
    font-size: 0.875rem;
  }
}

a {
  display: block;
  line-height: 1.5;
  text-decoration: none;

  &:hover {
    text-decoration: none;
  }
}
</style>
