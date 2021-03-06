<template>
  <div>
    <div class="shadow-lg rounded p-3">
      <!-- Palette menu -->
      <div v-if="!showModal">
        <div class="flex flex-wrap" v-if="palette.length > 0">
          <template v-for="(item, i) in palette">
            <div
              class="w-10 h-10 m-1 border border-gray-700 rounded-full cursor-pointer swatch"
              v-bind:style="{ backgroundColor: item['hex'] }"
              @click="makeActiveSwatch(i)"
            ></div>
          </template>
          <div
            class="w-10 h-10 m-1  cursor-pointer rounded-full flex items-center justify-center font-bold text-4xl text-red-700 hover:text-red-800"
            @click="addNewSwatch()"
          >
            +
          </div>
        </div>
        <div v-else class="block w-full">
          <span class="loader h-8 w-8 float-left mr-2"></span>Analyzing
          palette..
        </div>
        <!-- Preset Palette Selector -->
        <div class="mt-4">
          <label for="presetPalettes" class="mt-4">Preset Palettes</label>
          <div class="inline-block relative w-full">
            <select
              id="presetPalettes"
              v-model="presetPaletteSelection"
              @change="presetPaletteSelected"
              class="block appearance-none w-full bg-white border border-gray-300 hover:border-gray-500 px-4 py-2 pr-8 rounded leading-tight focus:outline-none focus:shadow-outline"
            >
              <option id="original" name="paletteColor" value="original"
                >Original</option
              >
              <option id="custom" name="paletteColor" value="custom" disabled
                >Custom</option
              >
              <template v-for="(p, i) in presetPalettes">
                <option :id="p.value" name="paletteColor" :value="p.value">{{
                  p.name
                }}</option>
              </template>
            </select>
            <div
              class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700"
            >
              <svg
                class="fill-current h-4 w-4"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
              >
                <path
                  d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"
                />
              </svg>
            </div>
          </div>
        </div>
      </div>
      <!-- Colorpicker Modal -->
      <div v-else>
        <div class="color-modal">
          <sketch-picker
            :value="palette[activeSwatch]"
            @input="storeCurrentColor"
          />
        </div>
        <div class="mt-2">
          <hr />
          <div class="inline-block w-full mt-2">
            <button class="btn-grey float-left" @click="showModal = !showModal">
              Cancel
            </button>
            <button class="btn-blue float-right mr-2" @click="selectColor">
              Select
            </button>
            <button
              class="btn-red float-right mr-2"
              @click="removeSwatch(activeSwatch)"
            >
              Remove
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Sketch } from 'vue-color'

export default {
  components: {
    'sketch-picker': Sketch
  },
  props: ['initialPalette'],
  data() {
    return {
      // numberOfColors: 4,

      currentColor: '', // for temporary holding swatch value from color-picker
      showModal: false,
      activeSwatch: 0,
      palette: [], // the colors as they are created by the color pickers
      convertedColorArray: [], // the colors as they are sent to the ditherer ([0,0,0])
      originalInitialPalette: [], //this holds the first initial palette loaded
      presetPaletteSelection: 'original',
      presetPalettes: [
        {
          name: 'Red',
          value: 'red',
          colors: [
            { hex: '#ffffff' },
            { hex: '#f46842' },
            { hex: '#aa2f0d' },
            { hex: '#000000' }
          ]
        },
        {
          name: 'Red Monochrome',
          value: 'redmono',
          colors: [{ hex: '#ffe3db' }, { hex: '#4f1403' }]
        },
        {
          name: 'Green Monochrome',
          value: 'greenmono',
          colors: [{ hex: '#eeffdb' }, { hex: '#1d3801' }]
        },
        {
          name: 'Green',
          value: 'green',
          colors: [
            { hex: '#ffffff' },
            { hex: '#c4f441' },
            { hex: '#6da90c' },
            { hex: '#000000' }
          ]
        },
        {
          name: 'Blue Monochrome',
          value: 'bluemono',
          colors: [{ hex: '#dbf9ff' }, { hex: '#02474f' }]
        },
        {
          name: 'Blue',
          value: 'blue',
          colors: [
            { hex: '#ffffff' },
            { hex: '#41e2f4' },
            { hex: '#0c9fa9' },
            { hex: '#000000' }
          ]
        },
        {
          name: 'Yellow Monochrome',
          value: 'yellowmono',
          colors: [{ hex: '#fffedb' }, { hex: '#303001' }]
        },
        {
          name: 'Yellow',
          value: 'yellow',
          colors: [
            { hex: '#ffffff' },
            { hex: '#f4eb41' },
            { hex: '#a9a40c' },
            { hex: '#000000' }
          ]
        },
        {
          name: 'Purple and Green',
          value: 'purplegreen',
          colors: [{ hex: '#76C066' }, { hex: '#AD2BBB' }]
        },
        {
          name: 'Yellow and Red',
          value: 'yellowred',
          colors: [{ hex: '#FFEE2C' }, { hex: '#E20023' }]
        },
        {
          name: 'Black & White',
          value: 'blackwhite',
          colors: [{ hex: '#ffffff' }, { hex: '#000000' }]
        },
        {
          name: 'RGBY',
          value: 'rgby',
          colors: [
            { hex: '#FF0000' },
            { hex: '#00FF00' },
            { hex: '#0000FF' },
            { hex: '#FFFF00' }
          ]
        },
        {
          name: 'Hamburger',
          value: 'hamburger',
          colors: [
            { hex: '#ca9a4a' },
            { hex: '#502902' },
            { hex: '#BB1313' },
            { hex: '#57BB2C' }
          ]
        },

        // { name: 'A Field', value: 'field', colors: [
        //     {hex: '#81b4ed'}, {hex: '#7cb8af'}, {hex: '#c6ba79'}, {hex: '#7f6000'}
        // ]},
        {
          name: 'Retrofuturism',
          value: 'retrofuturism',
          colors: [
            { hex: '#000000' },
            { hex: '#c90fff' },
            { hex: '#00cece' },
            { hex: '#ffb600' }
          ]
        },
        // { name: 'Rupaul\'s Drag Race', value: 'rupaul', colors: [
        //     {hex: '#f397d6'}, {hex: '#915741'}, {hex: '#7c0c0e'}, {hex: '#000000'}
        // ]},
        {
          name: 'Bonfire of the Vanities',
          value: 'bonfire',
          colors: [
            { hex: '#FFDE6C' },
            { hex: '#000000' },
            { hex: '#910c02' },
            { hex: '#567196' }
          ]
        }
      ]
    }
  },
  computed: {},
  watch: {
    initialPalette: function(newVal, oldVal) {
      // watch it
      // console.log('Prop changed: ', newVal, ' | was: ', oldVal)

      this.rgbToHex(newVal)
    }
  },
  methods: {
    // When a custom palette is selected
    presetPaletteSelected(e) {
      this.palette = []

      if (e.target.value == 'original') {
        this.originalInitialPalette.forEach((v) => {
          this.palette.push(v)
        })
      } else {
        this.presetPalettes.forEach((v) => {
          if (e.target.value == v.value) {
            v.colors.forEach((c) => {
              this.palette.push(c)
            })
          }
        })
      }

      this.updatePallete()
    },
    // When the user selects the color from the modal
    selectColor() {
      this.palette[this.activeSwatch] = { hex: this.currentColor['hex'] }
      this.updatePallete()
      this.showModal = false
    },
    // This holds the selected color until user hits Select in the color modal
    storeCurrentColor(color) {
      this.currentColor = color
    },
    // Determine if a swatch is the active one, this controls which classes display on that swatch when selected
    isActiveSwatch(i) {
      if (this.activeSwatch === i) {
        return true
      } else {
        return false
      }
    },
    // Make Active Swatch
    // When a swatch is clicked on, make it active and open the color picker
    makeActiveSwatch(i) {
      this.activeSwatch = i //tells the picker which swatch is being updated
      this.showModal = true

      this.presetPaletteSelection = 'custom'
    },
    // Add New Swatch
    // This adds a new color to the colors
    addNewSwatch() {
      this.palette.push({ hex: '#ffffff' })
      this.activeSwatch = this.palette.length
      this.presetPaletteSelection = 'custom'
      this.updatePallete()
    },
    // Remove a swatch and update the palette
    removeSwatch(i) {
      console.log(i)
      this.palette.splice(i, 1)
      this.updatePallete()
      this.showModal = false
    },
    // Update Colors
    // This updates the colors that are used by the actual ditherer
    // Requires an array of hex colors in the format [{hex: '#ff000'},{hex: '#ff0ff'}]
    updatePallete() {
      // update the colors, requires an array of
      var colorTuplesArray = []
      this.palette.forEach((v) => {
        colorTuplesArray.push(this.hexToRgb(v['hex']))
      })
      this.$emit('update-palette', colorTuplesArray)
    },
    //Convert Hex to RGB
    // Given a hex value, return the RGB tuple
    hexToRgb(hex) {
      var result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
      return result
        ? [
            parseInt(result[1], 16),
            parseInt(result[2], 16),
            parseInt(result[3], 16)
          ]
        : null
    },
    // Convert RGB To Hex
    // this converts an array of tupes into hex values
    rgbToHex() {
      this.palette = []
      var pal = this.initialPalette
      for (let i = 0; i < pal.length; i++) {
        var r = pal[i][0]
        var g = pal[i][1]
        var b = pal[i][2]
        var hex =
          '#' + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1)
        this.palette.push({ hex })

        if (this.originalInitialPalette.length < i) {
          //if there are less values in originalInitialPalette than colors processed, it means we need to store it
          this.originalInitialPalette.push({ hex })
        }
      }
    }
  }
}
</script>

<style>
.vc-sketch {
  width: auto !important;
  box-shadow: none !important;
}

.vc-sketch-alpha-wrap {
  display: none;
}

.vc-sketch-field--single:last-of-type {
  display: none;
}

.vc-sketch-presets {
  display: none;
}
.vc-sketch-hue-wrap {
  height: 24px !important;
}

.vc-hue-picker {
  margin-top: 0 !important;
  height: 24px !important;
}
</style>
