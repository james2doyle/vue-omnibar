<template>
  <transition name="omnibar-fade">
    <div class="omnibar" :class="{ 'omnibar-shadow': shadow, 'omnibar-overlay': overlay, 'omnibar-rounded': rounded }" @click="handleOverlayClick($event)" v-if="open">
      <div class="omnibar-inside">
        <slot name="header">
          <h3>Omnibar</h3>
        </slot>
        <input type="search" name="search" id="omnibar-search" class="omnibar-search" ref="omnibar-search" placeholder="Search" @input="handleInput($event)" @keyup="handleArrowKeys($event)" autocomplete="off" autofocus />
        <div class="omnibar-search-list" ref="omnibar-search-list" @keyup="handleArrowKeys($event)">
          <slot name="initial" v-bind="{ initial }" v-if="initial.length > 0 && results.length < 1">
            <div v-for="item in initial" :key="JSON.stringify(item)" class="omnibar-search-initial-item">
              <a href="#" @click.prevent="handleClick(item)" v-text="item"></a>
            </div>
          </slot>
          <slot name="results" v-bind="{ results }" v-else>
            <div v-for="item in results" :key="JSON.stringify(item)" class="omnibar-search-results-item">
              <a href="#" @click.prevent="handleClick(item)" v-text="item"></a>
            </div>
          </slot>
        </div>
      </div>
    </div>
  </transition>
</template>

<script lang="ts">
import Vue from 'vue';
import Fuse from 'fuse.js';

type ItemData = any | Record<string, any>

interface Data {
  search: string
  results: Array<ItemData>
  keys: Array<string> | boolean
  open: boolean
}

interface Methods {
  handleArrowKeys(e: KeyboardEvent): void
  handleKeyDown(e: KeyboardEvent): void
  handleKeyUp(e: KeyboardEvent): void
  openOmnibar(): void
  closeOmnibar(): void
  handleInput(e: KeyboardEvent): void
  handleOverlayClick(e: MouseEvent): void
  handleClick(item: ItemData): void
}

interface Computed {}

interface Props {
  keybinding: Array<string>
  shadow: boolean
  overlay: boolean
  rounded: boolean
  options: Fuse.IFuseOptions<string>
  data: Array<ItemData>
  initial: Array<ItemData>
}

// selector list for elements that can be focused
const FOCUSABLE = 'a, button, input, textarea, select, details, [tabindex]:not([tabindex="-1"])';

export default Vue.extend<Data, Methods, Computed, Props>({
  name: 'Omnibar',
  props: {
    keybinding: {
      type: Array,
      required: false,
      default: () => ['shift', 'p'] as Array<string>,
    },
    shadow: {
      type: Boolean,
      required: false,
      default: true,
    },
    overlay: {
      type: Boolean,
      required: false,
      default: true,
    },
    rounded: {
      type: Boolean,
      required: false,
      default: true,
    },
    options: {
      type: Object,
      required: false,
      default: () => {},
    },
    data: {
      type: [Object, Array],
      required: true,
    },
    initial: {
      type: [Object, Array],
      required: false,
      default: () => [],
    },
  },
  data() {
    return {
      search: '',
      results: [],
      keys: {},
      open: true, // false,
    };
  },
  beforeMount() {
    window.addEventListener('keydown', this.handleKeyDown);
    window.addEventListener('keyup', this.handleKeyUp);
  },
  beforeDestroy() {
    window.removeEventListener('keydown', this.handleKeyDown);
    window.removeEventListener('keyup', this.handleKeyUp);
  },
  methods: {
    handleArrowKeys(e: KeyboardEvent) {
      if (!this.$refs['omnibar-search-list']) {
        return;
      }

      const key = e.key;
      const target = e.target as HTMLElement;
      const currentTarget = e.currentTarget as HTMLElement;

      const items = Array.from((this.$refs['omnibar-search-list'] as HTMLElement).querySelectorAll(FOCUSABLE)) as Array<HTMLElement>;
      const currentIndex = items.indexOf(target);

      if (currentIndex === -1) {
        const firstElement = items[0];
        if (e.key === 'ArrowDown' && firstElement) {
          firstElement.focus();
        }

        return;
      }

      if (key === 'ArrowDown') {
        if (currentIndex + 1 === items.length) {
          return (this.$refs['omnibar-search'] as HTMLElement).focus();
        }

        items[currentIndex + 1].focus();
      }

      if (key === 'ArrowUp') {
        if (currentIndex - 1 === -1) {
          return (this.$refs['omnibar-search'] as HTMLElement).focus();
        }

        items[currentIndex - 1].focus();
      }
    },
    handleKeyDown(e: KeyboardEvent) {
      const key = e.key.toLowerCase() as typeof e.key;
      // if escape is pressed and the modal is open, and you are not focused on the search input or the input is blank
      if (key === 'escape' && this.open && (document.activeElement !== this.$refs['omnibar-search'] || this.search === '')) {
        this.closeOmnibar();
      }

      if (!e.isComposing && !e.repeat && this.keybinding.includes(key)) {
        this.keys[key] = true;
      }

      const allKeys = Object.values(this.keys).filter(Boolean).length === this.keybinding.length;
      if (allKeys && this.open === false) {
        this.openOmnibar();
      }
    },
    handleKeyUp(e: KeyboardEvent) {
      const key = e.key.toLowerCase() as typeof e.key;
      if (!e.isComposing && !e.repeat && this.keybinding.includes(key)) {
        this.keys[key] = false;
      }
    },
    openOmnibar() {
      this.open = true;
      // nextTick if after v-if transition is done
      this.$nextTick(() => {
        // timeout so that the keys being pressed are not captured in the focused input
        setTimeout(() => {
          (this.$refs['omnibar-search'] as HTMLElement).focus();
        });
      });
    },
    closeOmnibar() {
      this.open = false;
    },
    handleInput(e: KeyboardEvent) {
      this.search = (e.target as HTMLElement).value || '';
      const fuse = new Fuse(this.data, this.options);
      const indexes = fuse.search(this.search);

      this.results =
        indexes.length > 0
          ? indexes.map(({ refIndex }) => {
              return this.data[refIndex];
            })
          : [];
    },
    handleOverlayClick({ target }) {
      if ((target as HTMLElement).matches('.omnibar')) {
        this.closeOmnibar();
      }
    },
    handleClick(item: ItemData) {
      this.$emit('click', item);
      this.closeOmnibar();
    }
  },
});
</script>

<style>
.omnibar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
  width: 100%;
  height: 100%;
  padding: 1rem;
  box-sizing: border-box;
}
.omnibar-fade-enter-active, .omnibar-fade-leave-active {
  transition: opacity 200ms ease;
}
.omnibar-fade-enter, .omnibar-fade-leave-to {
  opacity: 0;
}
.omnibar.omnibar-overlay {
  background-color: rgba(0,0,0,0.5);
}
.omnibar.omnibar-shadow .omnibar-inside {
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
}
.omnibar.omnibar-rounded {
  border-radius: 5px;
}
.omnibar-inside {
  padding: 1rem 2rem 2rem;
  background-color: white;
  max-width: 24rem;
  margin: 10% auto 0;
}
.omnibar-search {
  width: 100%;
  margin-bottom: 1rem;
}
</style>
