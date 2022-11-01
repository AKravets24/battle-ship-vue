<template>
  <BattleShip/>
</template>

<script lang="ts">

import { Options, Vue } from 'vue-class-component';
import BattleShip from './pages/BattleShip.vue';

@Options({
  components: {
    BattleShip,
  },
})
export default class App extends Vue {}
</script>

