<template>
  <div id="app">
    <omnibar :data="data" :initial="data.slice(0, 4)" @click="handleClick">
      <h3 slot="header">Command Palette</h3>
      <template #initial="{ initial }">
        <div v-for="item in initial" :key="JSON.stringify(item)" class="omnibar-search-initial-list">
          <a href="#" @click.prevent="handleClick(item)" v-text="item"></a>
        </div>
      </template>
      <template #results="{ results }">
        <div v-for="item in results" :key="JSON.stringify(item)" class="omnibar-search-results-list">
          <a href="#" @click.prevent="handleClick(item)" v-text="item"></a>
        </div>
      </template>
    </omnibar>
    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import Omnibar from '@/omnibar.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    Omnibar
  },
  data() {
    return {
      data: [
        '1️⃣ --- one',
        '2️⃣ --- two',
        '3️⃣ --- three',
        '4️⃣ --- four',
        '5️⃣ --- five',
        '6️⃣ --- six',
        '7️⃣ --- seven',
        '8️⃣ --- eight',
        '9️⃣ --- nine',
        '🔟 --- ten',
      ]
    };
  },
  methods: {
    handleClick(item: any) {
      window.alert(item);
    }
  },
});
</script>

<style type="text/css">
.omnibar-search-list a {
  display: inline-block;
  width: 100%;
}
</style>
