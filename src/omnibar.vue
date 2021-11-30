<template>
  <transition name="omnibar-fade">
    <div class="omnibar" :ref="name" :data-name="name" :class="{ 'omnibar-shadow': shadow, 'omnibar-overlay': overlay }" @click="handleOverlayClick($event)" v-if="open">
      <div class="omnibar-inside">
        <slot name="header">
          <h3>Omnibar</h3>
        </slot>
        <input type="search" name="search" id="omnibar-search" class="omnibar-search" ref="omnibar-search" :placeholder="placeholder" @input="handleInput($event)" @keyup="handleArrowKeys($event)" autocomplete="off" autofocus />
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
  keys: Record<string, boolean>
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
  name: string | boolean | null
  keybinding: Array<string> | null
  shadow: boolean
  overlay: boolean
  options: Fuse.IFuseOptions<string>
  data: Array<ItemData>
  initial: Array<ItemData>
  placeholder: string
}

// selector list for elements that can be focused
const FOCUSABLE = 'a, button, input, textarea, select, details, [tabindex]:not([tabindex="-1"])';

export default Vue.extend<Data, Methods, Computed, Props>({
  name: 'Omnibar',
  props: {
    // namespace the modal so it can be opened/closed with a named event
    name: {
      type: [String, Boolean, null],
      required: false,
      default: '',
    },
    // which key combo to use to open the modal
    keybinding: {
      type: [Array, null],
      required: false,
      default: () => ['shift', 'p'] as Array<string>,
    },
    // add a shadow
    shadow: {
      type: Boolean,
      required: false,
      default: true,
    },
    // add an overlay
    overlay: {
      type: Boolean,
      required: false,
      default: true,
    },
    // additional options to pass to Fuse
    options: {
      type: Object,
      required: false,
      default: () => ({}),
    },
    // the list to search in
    data: {
      type: [Object, Array],
      required: true,
    },
    // the data to show when the modal opens
    initial: {
      type: [Object, Array],
      required: false,
      default: () => [],
    },
    // The search inputs placeholder
    placeholder: {
      type: String,
      required: false,
      default: 'Search',
    },
  },
  data() {
    return {
      search: '',
      results: [],
      keys: {},
      open: false,
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
  mounted() {
    // allow the modal to be opened/closed programmatically
    const evtOpen = Boolean(this.name) ? `openOmnibar.${this.name}` : 'openOmnibar';
    this.$root.$on(evtOpen, this.openOmnibar);
    const evtClose = Boolean(this.name) ? `closeOmnibar.${this.name}` : 'closeOmnibar';
    this.$root.$on(evtClose, this.closeOmnibar);
  },
  methods: {
    handleArrowKeys(e: KeyboardEvent) {
      const currentTarget = (this.$refs['omnibar-search-list'] as HTMLElement);

      if (!currentTarget) {
        return;
      }

      // prevent scrolling with space and arrow keys when fired on the omnibar
      if (["Space","ArrowUp","ArrowDown","ArrowLeft","ArrowRight"].indexOf(e.code) > -1) {
        e.preventDefault();
      }

      const key = e.key;
      const target = e.target as HTMLElement;

      const items = Array.from(currentTarget.querySelectorAll(FOCUSABLE)) as Array<HTMLElement>;
      const currentIndex = items.indexOf(target);

      if (currentIndex === -1) {
        const firstElement = items[0];
        if (e.key === 'ArrowDown' && firstElement) {
          firstElement.focus();
        }

        return;
      }

      const list = (this.$refs['omnibar-search'] as HTMLInputElement);

      if (key === 'ArrowDown') {
        if (currentIndex + 1 === items.length) {
          return list.focus();
        }

        items[currentIndex + 1].focus();
      }

      if (key === 'ArrowUp') {
        if (currentIndex - 1 === -1) {
          return list.focus();
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

      if (!e.isComposing && !e.repeat && (this.keybinding !== null && this.keybinding.includes(key))) {
        this.keys[key] = true;
      }

      const allKeys = this.keybinding !== null && Object.values(this.keys).filter(Boolean).length === this.keybinding.length;
      if (allKeys && this.open === false) {
        this.openOmnibar();
      }
    },
    handleKeyUp(e: KeyboardEvent) {
      const key = e.key.toLowerCase() as typeof e.key;
      if (!e.isComposing && !e.repeat && (this.keybinding !== null && this.keybinding.includes(key))) {
        this.keys[key] = false;
      }
    },
    openOmnibar() {
      this.open = true;
      // nextTick if after v-if transition is done
      this.$nextTick(() => {
        document.body.classList.add('omnibar-block-scrolling');
        // timeout so that the keys being pressed are not captured in the focused input
        setTimeout(() => {
          (this.$refs['omnibar-search'] as HTMLInputElement).focus();
        });
      });
    },
    closeOmnibar() {
      this.open = false;
      this.search = '';
      this.results = [];
      document.body.classList.remove('omnibar-block-scrolling');
    },
    handleInput(e: KeyboardEvent) {
      this.search = (e.target as HTMLInputElement).value || '';
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
body.omnibar-block-scrolling {
  overflow: hidden;
}
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
  overscroll-behavior: contain;
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
.omnibar-inside {
  padding: 1rem 2rem 2rem;
  background-color: white;
  max-width: 24rem;
  margin: 10% auto 0;
  position: relative;
  z-index: 2;
}
.omnibar-search {
  width: 100%;
  margin-bottom: 1rem;
}
</style>
