<template>
  <div class="ship-stats">
    <h3 class="title">{{ shipType }}</h3>

    <div v-for="(ship, i) in shipArr" :key="i" class="flexed justifyed base-font">
      <span class="span-prettier on-duty" v-if="ship.some(el => el.alive)">On duty</span>
      <span class="span-prettier poseidoned" v-else>Sunked</span>
      <span class="span-counter">{{ 100 - Math.round(damageCalc(shipsOwner, shipType, i) / ship.length * 100) }} </span>
    </div>

  </div>
</template>

<script lang="ts">

  import {defineComponent} from 'vue';

  export default defineComponent({

    name: "ShipStatuses",
    props: {
      shipArr: {type: Object, required: true},
      damageCalc: {type: Function, required: true},
      shipsOwner: {type: String, required: true},
      shipType: {type: String, required: true},
    },

  })
</script>

<style scoped lang="less">

  @baseFontSize: 20px;
  @smallFontSize: 15px;
  @largeFontSize: 25px;

  .base-font {
    font-size: @baseFontSize;
  }
  .small-font {
    font-size:  @smallFontSize;
  }

  .ship-stats {
    min-width: 120px;
    /*outline: 2px solid red;*/
  }

  .title {
    text-transform: capitalize;
    text-align: center;
  }

  .span-prettier {
    height: 25px;
    margin: 2px 0;
    min-width: 100px;
    text-align: center;
    border-radius: 3px;
  }

  .span-counter {
    width: 50px;
    text-align: center;
    height: 25px;
    margin: 2px 0;
  }

  .on-duty {
    background: lightgreen;
  }

  .poseidoned {
    background: firebrick;
  }

  .flexed {
    display: flex;
  }

  .justifyed {
    justify-content: space-around;
    flex-wrap: wrap;
  }

  .centred {
    justify-content: space-evenly;
    align-items: center;
  }

  .columned {
    flex-direction: column;
  }

</style>