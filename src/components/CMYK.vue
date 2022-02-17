<template>
  <div role="application" aria-label="Sketch color picker" :class="['vc-sketch', disableAlpha ? 'vc-sketch__disable-alpha' : '']">
    <div class="vc-cmyk-saturation-wrap">
      <saturation v-model="colors" @change="childChange"></saturation>
    </div>
    <div class="vc-cmyk-controls">
      <div class="vc-cmyk-sliders">
        <div class="vc-cmyk-hue-wrap">
          <hue v-model="colors" @change="childChange"></hue>
        </div>
      </div>
      <div class="vc-cmyk-color-wrap">
        <div :aria-label="`Current color is ${activeColor}`" class="vc-cmyk-active-color" :style="{background: activeColor}"></div>
        <checkboard></checkboard>
      </div>
    </div>
    <div class="vc-cmyk-field" v-if="!disableFields">
      <!-- cmyk -->
      <div class="vc-cmyk-field--single">
        <ed-in label="c" :value="cmyk.c" @change="inputChange"></ed-in>
      </div>
      <div class="vc-cmyk-field--single">
        <ed-in label="m" :value="cmyk.m" @change="inputChange"></ed-in>
      </div>
      <div class="vc-cmyk-field--single">
        <ed-in label="y" :value="cmyk.y" @change="inputChange"></ed-in>
      </div>
      <div class="vc-cmyk-field--single">
        <ed-in label="k" :value="cmyk.k" @change="inputChange"></ed-in>
      </div>
    </div>
  </div>
</template>

<script>
import colorMixin from '../mixin/color'
import editableInput from './common/EditableInput.vue'
import saturation from './common/Saturation.vue'
import hue from './common/Hue.vue'
import alpha from './common/Alpha.vue'
import checkboard from './common/Checkboard.vue'

const presetColors = [
  '#D0021B', '#F5A623', '#F8E71C', '#8B572A', '#7ED321',
  '#417505', '#BD10E0', '#9013FE', '#4A90E2', '#50E3C2',
  '#B8E986', '#000000', '#4A4A4A', '#9B9B9B', '#FFFFFF',
  'rgba(0,0,0,0)'
]

export default {
  name: 'CMYK',
  mixins: [colorMixin],
  components: {
    saturation,
    hue,
    alpha,
    'ed-in': editableInput,
    checkboard
  },
  props: {
    presetColors: {
      type: Array,
      default () {
        return presetColors
      }
    }
  },
  computed: {
    cmyk () {
      var t = this.colors.rgba

      var r = t.r / 255
      var g = t.g / 255
      var b = t.b / 255

      var k = Math.min(1 - r, 1 - g, 1 - b)
      var c = (1 - r - k) / (1 - k)
      var m = (1 - g - k) / (1 - k)
      var y = (1 - b - k) / (1 - k)

      c = Math.round(c * 100)
      m = Math.round(m * 100)
      y = Math.round(y * 100)
      k = Math.round(k * 100)

      return {c: c, m: m, y: y, k: k}
    },
    activeColor () {
      var rgba = this.colors.rgba
      return 'rgba(' + [rgba.r, rgba.g, rgba.b, rgba.a].join(',') + ')'
    }
  },
  methods: {
    handlePreset (c) {
      this.colorChange({
        hex: c,
        source: 'hex'
      })
    },
    childChange (data) {
      this.colorChange(data)
    },
    CMYK (cmyk) {
      var c = cmyk.c / 100
      var m = cmyk.m / 100
      var y = cmyk.y / 100
      var k = cmyk.k / 100

      var r = 1 - Math.min(1, c * (1 - k) + k)
      var g = 1 - Math.min(1, m * (1 - k) + k)
      var b = 1 - Math.min(1, y * (1 - k) + k)

      this.colorChange({
        r: r * 255 || this.colors.rgba.r,
        g: g * 255 || this.colors.rgba.g,
        b: b * 255 || this.colors.rgba.b,
        a: 1,
        source: 'rgba'
      })
    },
    inputChange (data) {
      if (!data) {
        return
      }
      var c = this.cmyk
      for (var k in data) {
        var v = parseFloat(data[k])
        if (v > 100) v = 100
        else if (v < 0) v = 0
        c[k] = v
      }
      this.CMYK(c)
    }
  }
}
</script>

<style>
.vc-cmyk {
  position: relative;
  width: 200px;
  padding: 10px 10px 0;
  box-sizing: initial;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 0 0 1px rgba(0, 0, 0, .15), 0 8px 16px rgba(0, 0, 0, .15);
}

.vc-cmyk-saturation-wrap {
  width: 100%;
  padding-bottom: 75%;
  position: relative;
  overflow: hidden;
}

.vc-cmyk-controls {
  display: flex;
}

.vc-cmyk-sliders {
  padding: 4px 0;
  flex: 1;
}

.vc-cmyk-sliders .vc-hue,
.vc-cmyk-sliders .vc-alpha-gradient {
  border-radius: 2px;
}

.vc-cmyk-hue-wrap {
  position: relative;
  height: 10px;
}

.vc-cmyk-alpha-wrap {
  position: relative;
  height: 10px;
  margin-top: 4px;
  overflow: hidden;
}

.vc-cmyk-color-wrap {
  width: 24px;
  height: 24px;
  position: relative;
  margin-top: 4px;
  margin-left: 4px;
  border-radius: 3px;
}

.vc-cmyk-active-color {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 2px;
  box-shadow: inset 0 0 0 1px rgba(0, 0, 0, .15), inset 0 0 4px rgba(0, 0, 0, .25);
  z-index: 2;
}

.vc-cmyk-color-wrap .vc-checkerboard {
  background-size: auto;
}

.vc-cmyk-field {
  display: flex;
  padding-top: 4px;
}

.vc-cmyk-field .vc-input__input {
  width: 90%;
  padding: 4px 0 3px 10%;
  border: none;
  box-shadow: inset 0 0 0 1px #ccc;
  font-size: 10px;
}

.vc-cmyk-field .vc-input__label {
  display: block;
  text-align: center;
  font-size: 11px;
  color: #222;
  padding-top: 3px;
  padding-bottom: 4px;
  text-transform: capitalize;
}

.vc-cmyk-field--single {
  flex: 1;
  padding-left: 6px;
}
.vc-cmyk-field--single:first-child {
  padding-left: 0;
}

.vc-cmyk-field--double {
  flex: 2;
}

.vc-cmyk__disable-alpha .vc-cmyk-color-wrap {
  height: 10px;
}
</style>
