<template>
  <div class="gardenLayout" :style="gardenStyles">
    <VueDragResize
      v-for="plant in plants"
      :key="plant.id"
      :ref="plant.id"
      class="plant"
      :isResizable="false"
      :parentLimitation="true"
      :w="plantScale(plant.radius)"
      :h="plantScale(plant.radius)"
      :x="plantScale(plant.position_x)"
      :y="plantScale(plant.position_y)"
      :stickSize="0"
      :style="plantBorderRadius(plant)"
      @clicked="selectPlant(plant.id)"
      @dragstop="plantMoved"
      :snapToGrid="true"
      :gridX="gardenScale"
      :gridY="gardenScale"
      :parentW="plantScale(gardenSize.width)"
      :parentH="plantScale(gardenSize.height)"
    >
    </VueDragResize>
  </div>
</template>

<script>
import VueDragResize from "vue-drag-resize";
import {
  collisionCheckHorizontal,
  collisionCheckVertical,
} from "../helpers/collisionCheckers";

export default {
  name: "gardenLayout",
  data() {
    return {
      gardenScale: 20,
      moveStart: {
        left: 0,
        top: 0,
        width: 0,
        height: 0,
      },
    };
  },
  props: {
    gardenSize: {
      validator: (info) => {
        return info.width && info.height;
      },
      required: true,
    },
    plants: {
      validator: (list) =>
        list.every(
          (plant) =>
            typeof plant.position_x === "number" &&
            typeof plant.position_y === "number" &&
            !!plant.radius &&
            !!plant.id
        ),
      required: true,
    },
    selected: {
      type: Number,
      required: true,
    },
  },
  computed: {
    gardenStyles: function () {
      const height = this.plantScale(this.gardenSize.height);
      const width = this.plantScale(this.gardenSize.width);
      return { height: `${height}px`, width: `${width}px` };
    },
  },
  methods: {
    plantBorderRadius: function ({ plant, radius }) {
      if (!plant) return {};
      const styles = {
        "border-radius": `${this.plantScale(radius) / 2}px`,
        "background-image": `url(${plant.photo_url})`,
        "background-size": "contain",
      };
      return styles;
    },
    plantScale: function (num) {
      return num * this.gardenScale;
    },
    selectPlant: function (id) {
      this.moveStart = this.$refs[id][0].rect;
      this.$emit("update:selected", id);
    },
    // On drag stop
    plantMoved: function (loc) {
      let i = 0;
      let collide = false;
      this.$log.info("LOC", loc);
      while (i < this.plants.length) {
        if (this.plants[i].id !== this.selected) {
          if (
            collisionCheckHorizontal(loc, this.plants[i], this.gardenScale) &&
            collisionCheckVertical(loc, this.plants[i], this.gardenScale)
          ) {
            collide = true;
          }
        }
        i++;
      }
      // No Collision
      const plantInfo = {
        position_y: loc.top / this.gardenScale,
        position_x: loc.left / this.gardenScale,
      };
      this.$log.info("NO COLLISION");
      this.$log.info("Plant Info", plantInfo);

      this.$emit("plant-moved", plantInfo);
      if (collide) {
        // Collision
        this.$log.info("COLLISION DETECTED");
        this.$buefy.toast.open({
          duration: 1000,
          message: "Two plants are overlapping, please move them",
          type: "is-danger",
        });
      }
    },
  },
  components: {
    VueDragResize,
  },
};
</script>

<style lang="sass">
.gardenLayout
  position: relative
  background-color: $brown
  margin: 10px
  border-radius: 10px
.plant
  background-color: $green
</style>
