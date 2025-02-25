<template>
  <div class="hu-color-picker" :class="{ light: isLightTheme }" :style="{ width: totalWidth + 'px' }">
    <div class="color-set">
      <Saturation
          ref="saturation"
          :color="rgbString"
          :hsv="hsv"
          :size="hueHeight"
          @selectSaturation="selectSaturation"
      />
      <Hue
          ref="hue"
          :hsv="hsv"
          :width="hueWidth"
          :height="hueHeight"
          @selectHue="selectHue"
      />
      <Alpha
          v-if="!hideAlpha"
          ref="alpha"
          :color="rgbString"
          :rgba="rgba"
          :width="hueWidth"
          :height="hueHeight"
          @selectAlpha="selectAlpha"
      />
    </div>
    <div :style="{ height: previewHeight + 'px' }" class="color-show">
      <Preview style="width: 100%;" :color="rgbaString" :height="previewHeight"/>
      <Sucker
          v-if="!suckerHide"
          :sucker-canvas="suckerCanvas"
          :sucker-area="suckerArea"
          @openSucker="openSucker"
          @selectSucker="selectSucker"/>
    </div>
    <Box v-if="!hideHex" name="HEX" :color="modelHex" @inputColor="inputHex"/>
    <Box v-if="!hideRgba" name="RGBA" :color="modelRgba" @inputColor="inputRgba"/>
    <Colors :color="rgbaString" :colors-default="colorsDefault" @selectColor="selectColor"/>
  </div>
</template>

<script>
import mixin from './mixin'
import Saturation from './Saturation.vue'
import Hue from './Hue.vue'
import Alpha from './Alpha.vue'
import Preview from './Preview.vue'
import Sucker from './Sucker.vue'
import Box from './Box.vue'
import Colors from './Colors.vue'

export default {
  components: {
    Saturation,
    Hue,
    Alpha,
    Preview,
    Sucker,
    Box,
    Colors
  },
  mixins: [mixin],
  props: {
    value: {
      type: [String, Object],
      default: '#000000',
    },
    theme: {
      type: String,
      default: 'dark'
    },
    suckerHide: {
      type: Boolean,
      default: true
    },
    suckerCanvas: {
      type: null, // HTMLCanvasElement
      default: null
    },
    suckerArea: {
      type: Array,
      default: () => []
    },
    colorsDefault: {
      type: Array,
      default: () => [
        '#000000', '#FFFFFF', '#FF1900', '#F47365', '#FFB243', '#FFE623', '#6EFF2A', '#1BC7B1',
        '#00BEFF', '#2E81FF', '#5D61FF', '#FF89CF', '#FC3CAD', '#BF3DCE', '#8E00A7', 'rgba(0,0,0,0)'
      ]
    },
    hideHex: {
      type: Boolean,
      default: false,
    },
    hideRgba: {
      type: Boolean,
      default: false,
    },
    hideAlpha: {
      type: Boolean,
      default: false,
    },
    enableAlpha: Boolean,
  },
  data() {
    return {
      hueWidth: 15,
      hueHeight: 152,
      previewHeight: 30,
      modelRgba: '',
      modelHex: '',
      r: 0,
      g: 0,
      b: 0,
      a: 1,
      h: 0,
      s: 0,
      v: 0
    }
  },
  computed: {
    isLightTheme() {
      return this.theme === 'light'
    },
    totalWidth() {
      return this.hueHeight + (this.hueWidth + 8) * 2
    },
    previewWidth() {
      return this.totalWidth - (this.suckerHide ? 0 : this.previewHeight)
    },
    rgba() {
      return {
        r: this.r,
        g: this.g,
        b: this.b,
        a: this.a
      }
    },
    hsv() {
      return {
        h: this.h,
        s: this.s,
        v: this.v
      }
    },
    rgbString() {
      return `rgb(${this.r}, ${this.g}, ${this.b})`
    },
    rgbaStringShort() {
      return `${this.r}, ${this.g}, ${this.b}, ${this.a}`
    },
    rgbaString() {
      return `rgba(${this.rgbaStringShort})`
    },
    hexString() {
      if ( this.enableAlpha && this.rgba.a === 0 ) {
        return 'transparent';
      }

      return this.rgb2hex(this.rgba, true)
    }
  },
  created() {
    this.updateColor(this.value);
    this.setText()
  },
  watch: {
    value(newValue) {
      this.updateColor(newValue);
    },
    rgba() {
      this.$emit('input', {
        rgba: this.rgba,
        hsv: this.hsv,
        hex: this.modelHex
      });
    }
  },
  methods: {
    updateColor(value) {
      let hexColor = '#000000';
      if (typeof value === 'object' && value) {
        hexColor = value.hex;
      } else if (typeof value === 'string') {
        hexColor = value;
      }

      if ( hexColor === this.hexString) {
        return;
      }

      Object.assign(this, this.setColorValue(hexColor));
    },
    selectSaturation(color) {
      const {r, g, b, h, s, v} = this.setColorValue(color)
      Object.assign(this, {r, g, b, h, s, v})
      this.setText()
    },
    selectHue(color) {
      const {r, g, b, h, s, v} = this.setColorValue(color)
      Object.assign(this, {r, g, b, h, s, v})
      this.setText()
      this.$nextTick(() => {
        this.$refs.saturation.renderColor()
        this.$refs.saturation.renderSlide()
      })
    },
    selectAlpha(a) {
      this.a = a
      this.setText()
    },
    inputHex(color) {
      const {r, g, b, a, h, s, v} = this.setColorValue(color)
      Object.assign(this, {r, g, b, a, h, s, v})
      this.modelHex = color
      this.modelRgba = this.rgbaStringShort
      this.$nextTick(() => {
        this.$refs.saturation.renderColor()
        this.$refs.saturation.renderSlide()
        this.$refs.hue.renderSlide()
      })
    },
    inputRgba(color) {
      const {r, g, b, a, h, s, v} = this.setColorValue(color)
      Object.assign(this, {r, g, b, a, h, s, v})
      this.modelHex = this.hexString
      this.modelRgba = color
      this.$nextTick(() => {
        this.$refs.saturation.renderColor()
        this.$refs.saturation.renderSlide()
        this.$refs.hue.renderSlide()
      })
    },
    setText() {
      this.modelHex = this.hexString
      this.modelRgba = this.rgbaStringShort
    },
    openSucker(isOpen) {
      this.$emit('openSucker', isOpen)
    },
    selectSucker(color) {
      const {r, g, b, a, h, s, v} = this.setColorValue(color)
      Object.assign(this, {r, g, b, a, h, s, v})
      this.setText()
      this.$nextTick(() => {
        this.$refs.saturation.renderColor()
        this.$refs.saturation.renderSlide()
        this.$refs.hue.renderSlide()
      })
    },
    selectColor(color) {
      const {r, g, b, a, h, s, v} = this.setColorValue(color)
      Object.assign(this, {r, g, b, a, h, s, v})
      this.setText()
      this.$nextTick(() => {
        this.$refs.saturation.renderColor()
        this.$refs.saturation.renderSlide()
        this.$refs.hue.renderSlide()
      })
    }
  }
}
</script>

<style lang="scss">
.hu-color-picker {
  position: absolute;
  padding: 10px;
  background: #1d2024;
  border-radius: 4px;
  box-shadow: 0 0 16px 0 rgba(0, 0, 0, 0.16);
  z-index: 1;

  &.light {
    background: #f7f8f9;

    .color-show {
      .sucker {
        background: #eceef0;
      }
    }

    .color-type {
      .name {
        background: #e7e8e9;
      }

      .value {
        color: #666;
        background: #eceef0;
      }
    }

    .colors.history {
      border-top: 1px solid #eee;
    }
  }

  canvas {
    vertical-align: top;
  }

  .color-set {
    display: flex;
  }

  .color-show {
    margin-top: 8px;
    display: flex;
  }
}
</style>
