<template>
  <div @click="moveWest(this.componentType)">
  </div>
  <component v-else :hasX=this.hasX :hasY=this.hasY :is="this.componentType"></component>
</template>

<script>
import CellFree from "@/components/Cells/CellFree";
import CellPlayer from "@/components/Cells/CellPlayer";
import SmallSnowball from "@/components/Cells/SimpleSnowballs/SmallSnowball";
import MediumSnowball from "@/components/Cells/SimpleSnowballs/MediumSnowball";
import BigSnowball from "@/components/Cells/SimpleSnowballs/BigSnowball";
import MediumAndBig from "@/components/Cells/ComposedSnowballs/MediumAndBig";
import SmallAndBig from "@/components/Cells/ComposedSnowballs/SmallAndBig";
import SmallAndMedium from "@/components/Cells/ComposedSnowballs/SmallAndMedium";
import SmallAndMediumAndBig from "@/components/Cells/ComposedSnowballs/SmallAndMediumAndBig";

export default {
  name: "CellWestPlayer",
  components: {
    'CellFree': CellFree,
    'CellPlayer': CellPlayer,
    'SmallSnowball': SmallSnowball,
    'MediumSnowball': MediumSnowball,
    'BigSnowball': BigSnowball,
    'MediumAndBig': MediumAndBig,
    'SmallAndBig': SmallAndBig,
    'SmallAndMedium': SmallAndMedium,
    'SmallAndMediumAndBig': SmallAndMediumAndBig,
  },
  props: ["componentType","hasX", "hasY"],
  data(){
    return{

    }
  },
  methods: {
    moveWest: function(compType){
      this.$emit('move-event',  ["hasWeast", compType]);
    }
  },
}
</script>

<style scoped>
div {
  background-size: cover;
  background-image: url("../../../assets/left.png");
  width: 60px;
  height: 60px;
  z-index: 2;
  position: absolute;
}
</style>
