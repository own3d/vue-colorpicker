<template>
  <div>
    <ul class="colors">
      <li v-for="item in colorsDefault" :key="item" class="item" @click="selectColor(item)">
        <div :style="{ background: `url(${imgAlphaBase64})` }" class="alpha"/>
        <div :style="{ background: item }" class="color"/>
      </li>
    </ul>

    <ul v-if="colorsHistory.length" class="colors">
      <li v-for="item in colorsHistory" :key="item" class="item" @click="selectColor(item)">
        <div :style="{ background: `url(${imgAlphaBase64})` }" class="alpha"/>
        <div :style="{ background: item }" class="color"/>
      </li>
    </ul>
  </div>
</template>

<script>
import mixin from './mixin'
export default {
  mixins: [mixin],
  props: {
    color: {
      type: String,
      default: '#000000'
    },
    colorsDefault: {
      type: Array,
      default: () => []
    },
    colorsHistoryKey: {
      type: String,
      default: ''
    },
    history: {
      type: Array,
      default: () => [],
    }
  },
  data() {
    return {
      imgAlphaBase64: '',
      colorsHistory: this.history,
    };
  },
  created() {
    this.imgAlphaBase64 = this.createAlphaSquare(4).toDataURL()
  },
  destroyed() {
    this.setColorsHistory(this.color)
  },
  methods: {
    selectColor(color) {
      this.$emit('selectColor', color)
    },
    setColorsHistory(color) {
      if (!color) {
        return
      }

      const colors = this.colorsHistory
      const index = colors.indexOf(color)

      if (index >= 0) {
        colors.splice(index, 1)
      }

      if (colors.length >= 8) {
        colors.length = 7
      }

      colors.unshift(color)
      this.colorsHistory = colors
    }
  }
}
</script>

<style lang="scss" scoped>
ul.colors, ul.colors li {
  margin: 0 !important;
  padding: 0 !important;
}

ul.colors {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-top: 1rem !important;
}



.colors {
  padding: 0;
  margin: 0;

  &.history {
    margin-top: 10px;
    border-top: 1px solid #2e333a;
  }

  .item {
    position: relative;
    width: 16px;
    height: 16px;
    border-radius: 3px;
    box-sizing: border-box;
    vertical-align: top;
    display: inline-block;
    transition: all 0.1s;
    cursor: pointer;

    &:hover {
      transform: scale(1.4);
    }

    .alpha {
      height: 100%;
      border-radius: 4px;
    }

    .color {
      position: absolute;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      border-radius: 3px;
    }
  }
}
</style>
