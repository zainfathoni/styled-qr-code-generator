<script setup lang="ts">
import StyledQRCode from '@/components/StyledQRCode.vue'
import {
  copyImageToClipboard,
  downloadPngElement,
  downloadSvgElement
} from '@/utils/convertToImage'
import type { CornerDotType, CornerSquareType, DotType } from 'qr-code-styling'
import { computed, ref, watch } from 'vue'
import 'vue-i18n'
import { getNumericCSSValue } from './utils/formatting'
import { sortedLocales } from './utils/language'
import { defaultPreset, allPresets } from './utils/presets'

const data = ref(defaultPreset.data)
const image = ref(defaultPreset.image)
const width = ref(defaultPreset.width)
const height = ref(defaultPreset.height)
const margin = ref(defaultPreset.margin)
const imageMargin = ref(defaultPreset.imageOptions.margin)

const dotsOptionsColor = ref(defaultPreset.dotsOptions.color)
const dotsOptionsType = ref(defaultPreset.dotsOptions.type)
const cornersSquareOptionsColor = ref(defaultPreset.cornersSquareOptions.color)
const cornersSquareOptionsType = ref(defaultPreset.cornersSquareOptions.type)
const cornersDotOptionsColor = ref(defaultPreset.cornersDotOptions.color)
const cornersDotOptionsType = ref(defaultPreset.cornersDotOptions.type)
const styleBorderRadius = ref(getNumericCSSValue(defaultPreset.style.borderRadius as string))
const styledBorderRadiusFormatted = computed(() => `${styleBorderRadius.value}px`)
const styleBackground = ref(defaultPreset.style.background)

const dotsOptions = computed(() => ({
  color: dotsOptionsColor.value,
  type: dotsOptionsType.value
}))
const cornersSquareOptions = computed(() => ({
  color: cornersSquareOptionsColor.value,
  type: cornersSquareOptionsType.value
}))
const cornersDotOptions = computed(() => ({
  color: cornersDotOptionsColor.value,
  type: cornersDotOptionsType.value
}))
const style = computed(() => ({
  borderRadius: styledBorderRadiusFormatted.value,
  background: styleBackground.value
}))
const imageOptions = computed(() => ({
  margin: imageMargin.value
}))

const qrCodeProps = computed(() => ({
  data: data.value,
  image: image.value,
  width: width.value,
  height: height.value,
  margin: margin.value,
  dotsOptions: dotsOptions.value,
  cornersSquareOptions: cornersSquareOptions.value,
  cornersDotOptions: cornersDotOptions.value,
  imageOptions: imageOptions.value
}))

/* random settings utils */

function createRandomColor() {
  return '#' + Math.floor(Math.random() * 16777215).toString(16)
}

function getRandomItemInArray(array: any[]) {
  return array[Math.floor(Math.random() * array.length)]
}

function randomizeStyleSettings() {
  const dotTypes: DotType[] = [
    'dots',
    'rounded',
    'classy',
    'classy-rounded',
    'square',
    'extra-rounded'
  ]
  const cornerSquareTypes: CornerSquareType[] = ['dot', 'square', 'extra-rounded']
  const cornerDotTypes: CornerDotType[] = ['dot', 'square']

  dotsOptionsType.value = getRandomItemInArray(dotTypes)
  dotsOptionsColor.value = createRandomColor()

  cornersSquareOptionsType.value = getRandomItemInArray(cornerSquareTypes)
  cornersSquareOptionsColor.value = createRandomColor()

  cornersDotOptionsType.value = getRandomItemInArray(cornerDotTypes)
  cornersDotOptionsColor.value = createRandomColor()

  styleBackground.value = createRandomColor()
}

const selectedPreset = ref(defaultPreset)

watch(selectedPreset, () => {
  data.value = selectedPreset.value.data
  image.value = selectedPreset.value.image
  width.value = selectedPreset.value.width
  height.value = selectedPreset.value.height
  margin.value = selectedPreset.value.margin
  imageMargin.value = selectedPreset.value.imageOptions.margin
  dotsOptionsColor.value = selectedPreset.value.dotsOptions.color
  dotsOptionsType.value = selectedPreset.value.dotsOptions.type
  cornersSquareOptionsColor.value = selectedPreset.value.cornersSquareOptions.color
  cornersSquareOptionsType.value = selectedPreset.value.cornersSquareOptions.type
  cornersDotOptionsColor.value = selectedPreset.value.cornersDotOptions.color
  cornersDotOptionsType.value = selectedPreset.value.cornersDotOptions.type
  styleBorderRadius.value = getNumericCSSValue(selectedPreset.value.style.borderRadius as string)
  styleBackground.value = selectedPreset.value.style.background
})

/* export image utils */
const options = computed(() => ({
  width: width.value,
  height: height.value
}))

async function copyQRToClipboard() {
  console.debug('Copying image to clipboard')
  const qrCode = document.querySelector('#qr-code-container')
  if (qrCode) {
    await copyImageToClipboard(qrCode as HTMLElement, options.value)
  }
}

function downloadQRImageAsPng() {
  console.debug('Copying image to clipboard')
  const qrCode = document.querySelector('#qr-code-container')
  if (qrCode) {
    downloadPngElement(qrCode as HTMLElement, 'qr-code.png', options.value)
  }
}

function downloadQRImageAsSvg() {
  console.debug('Copying image to clipboard')
  const qrCode = document.querySelector('#qr-code-container')
  if (qrCode) {
    downloadSvgElement(qrCode as HTMLElement, 'qr-code.svg', options.value)
  }
}

function saveQRConfig() {
  console.debug('Saving QR code config')
  const qrCodeConfig = {
    props: qrCodeProps.value,
    style: style.value
  }
  const qrCodeConfigString = JSON.stringify(qrCodeConfig)
  const qrCodeConfigBlob = new Blob([qrCodeConfigString], { type: 'application/json' })
  const qrCodeConfigUrl = URL.createObjectURL(qrCodeConfigBlob)
  const qrCodeConfigLink = document.createElement('a')
  qrCodeConfigLink.href = qrCodeConfigUrl
  qrCodeConfigLink.download = 'qr-code-config.json'
  qrCodeConfigLink.click()
}

function loadQrConfig() {
  console.debug('Loading QR code config')
  const qrCodeConfigInput = document.createElement('input')
  qrCodeConfigInput.type = 'file'
  qrCodeConfigInput.accept = 'application/json'
  qrCodeConfigInput.onchange = (event: Event) => {
    const target = event.target as HTMLInputElement
    if (target.files) {
      const file = target.files[0]
      const reader = new FileReader()
      reader.onload = (event: ProgressEvent<FileReader>) => {
        const target = event.target as FileReader
        const result = target.result as string
        const qrCodeConfig = JSON.parse(result)
        const qrCodeProps = qrCodeConfig.props
        const qrCodeStyle = qrCodeConfig.style
        data.value = qrCodeProps.data
        image.value = qrCodeProps.image
        width.value = qrCodeProps.width
        height.value = qrCodeProps.height
        margin.value = qrCodeProps.margin
        dotsOptionsColor.value = qrCodeProps.dotsOptions.color
        dotsOptionsType.value = qrCodeProps.dotsOptions.type
        cornersSquareOptionsColor.value = qrCodeProps.cornersSquareOptions.color
        cornersSquareOptionsType.value = qrCodeProps.cornersSquareOptions.type
        cornersDotOptionsColor.value = qrCodeProps.cornersDotOptions.color
        cornersDotOptionsType.value = qrCodeProps.cornersDotOptions.type
        styleBorderRadius.value = qrCodeStyle.borderRadius
        styleBackground.value = qrCodeStyle.background
      }
      reader.readAsText(file)
    }
  }
  qrCodeConfigInput.click()
}

function uploadImage() {
  console.debug('Uploading image')
  const imageInput = document.createElement('input')
  imageInput.type = 'file'
  imageInput.accept = 'image/*'
  imageInput.onchange = (event: Event) => {
    const target = event.target as HTMLInputElement
    if (target.files) {
      const file = target.files[0]
      const reader = new FileReader()
      reader.onload = (event: ProgressEvent<FileReader>) => {
        const target = event.target as FileReader
        const result = target.result as string
        image.value = result
      }
      reader.readAsDataURL(file)
    }
  }
  imageInput.click()
}
</script>

<template>
  <main class="relative grid place-items-center px-6 py-20 sm:p-8" role="main">
    <div class="absolute end-4 top-4 flex flex-row items-center gap-4">
      <form class="flex flex-row items-center">
        <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24">
          <g
            fill="none"
            stroke="#abcbca"
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
          >
            <path d="M4 5h7M7 4c0 4.846 0 7 .5 8" />
            <path
              d="M10 8.5c0 2.286-2 4.5-3.5 4.5S4 11.865 4 11c0-2 1-3 3-3s5 .57 5 2.857c0 1.524-.667 2.571-2 3.143m2 6l4-9l4 9m-.9-2h-6.2"
            />
          </g>
        </svg>
        <select
          class="secondary-button cursor-pointer text-center"
          id="locale-select"
          v-model="$i18n.locale"
        >
          <option v-for="(locale, index) in sortedLocales" :key="index" :value="locale">
            {{ $t(locale) }}
          </option>
        </select>
      </form>
      <div class="vertical-border"></div>
      <a class="icon-button" href="https://github.com/lyqht/styled-qr-code-generator">
        <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24">
          <path
            fill="#abcbca"
            d="M12.001 2c-5.525 0-10 4.475-10 10a9.994 9.994 0 0 0 6.837 9.488c.5.087.688-.213.688-.476c0-.237-.013-1.024-.013-1.862c-2.512.463-3.162-.612-3.362-1.175c-.113-.288-.6-1.175-1.025-1.413c-.35-.187-.85-.65-.013-.662c.788-.013 1.35.725 1.538 1.025c.9 1.512 2.337 1.087 2.912.825c.088-.65.35-1.087.638-1.337c-2.225-.25-4.55-1.113-4.55-4.938c0-1.088.387-1.987 1.025-2.688c-.1-.25-.45-1.275.1-2.65c0 0 .837-.262 2.75 1.026a9.28 9.28 0 0 1 2.5-.338c.85 0 1.7.112 2.5.337c1.913-1.3 2.75-1.024 2.75-1.024c.55 1.375.2 2.4.1 2.65c.637.7 1.025 1.587 1.025 2.687c0 3.838-2.337 4.688-4.563 4.938c.363.312.676.912.676 1.85c0 1.337-.013 2.412-.013 2.75c0 .262.188.574.688.474A10.016 10.016 0 0 0 22 12c0-5.525-4.475-10-10-10Z"
          />
        </svg>
      </a>
    </div>
    <div class="w-full md:w-5/6">
      <div class="mb-8 flex w-full flex-col items-center justify-center">
        <h1 class="text-4xl">{{ $t('Styled QR Code Generator') }}</h1>
      </div>
      <div class="flex flex-col-reverse items-start justify-center gap-4 md:flex-row md:gap-12">
        <div
          id="main-content"
          class="flex w-full shrink-0 flex-col items-center justify-center md:w-fit"
        >
          <div id="qr-code-container">
            <div
              class="grid place-items-center overflow-hidden"
              :style="[
                style,
                {
                  width: '200px',
                  height: '200px'
                }
              ]"
            >
              <StyledQRCode
                v-if="data"
                v-bind="{ ...qrCodeProps, width: 200, height: 200 }"
                role="img"
                aria-label="QR code"
              />
              <p v-else>{{ $t('No data!') }}</p>
            </div>
          </div>
          <div class="mt-4 flex flex-col items-center gap-2">
            <div class="flex flex-col items-center justify-center gap-3">
              <button
                id="copy-qr-image-button"
                class="button flex w-fit flex-row gap-1"
                @click="copyQRToClipboard"
                :aria-label="$t('Copy QR Code to clipboard')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <g
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  >
                    <path
                      d="M8 10a2 2 0 0 1 2-2h8a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2h-8a2 2 0 0 1-2-2z"
                    />
                    <path d="M16 8V6a2 2 0 0 0-2-2H6a2 2 0 0 0-2 2v8a2 2 0 0 0 2 2h2" />
                  </g>
                </svg>
                <p>{{ $t('Copy QR Code to clipboard') }}</p>
              </button>
              <button
                id="save-qr-code-config-button"
                class="button flex w-fit flex-row gap-1"
                @click="saveQRConfig"
                :aria-label="$t('Save QR Code configuration')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <g
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  >
                    <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                    <path
                      d="M17 21H7a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h7l5 5v11a2 2 0 0 1-2 2zm-5-4v-6"
                    />
                    <path d="M9.5 14.5L12 17l2.5-2.5" />
                  </g>
                </svg>
                <p>{{ $t('Save QR Code configuration') }}</p>
              </button>
              <button
                id="load-qr-code-config-button"
                class="button flex w-fit flex-row gap-1"
                @click="loadQrConfig"
                :aria-label="$t('Load QR Code configuration')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <g
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  >
                    <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                    <path
                      d="M17 21H7a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h7l5 5v11a2 2 0 0 1-2 2zm-5-10v6"
                    />
                    <path d="M9.5 13.5L12 11l2.5 2.5" />
                  </g>
                </svg>
                <p>{{ $t('Load QR Code configuration') }}</p>
              </button>
            </div>
            <div id="export-options" class="pt-4">
              <p class="pb-2">{{ $t('Export as') }}</p>
              <div class="flex flex-row items-center gap-2">
                <button
                  id="download-qr-image-button-png"
                  class="button"
                  @click="downloadQRImageAsPng"
                  :aria-label="$t('Download QR Code as PNG')"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    viewBox="0 0 24 24"
                  >
                    <g
                      fill="none"
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                    >
                      <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                      <path
                        d="M5 12V5a2 2 0 0 1 2-2h7l5 5v4m1 3h-1a2 2 0 0 0-2 2v2a2 2 0 0 0 2 2h1v-3M5 18h1.5a1.5 1.5 0 0 0 0-3H5v6m6 0v-6l3 6v-6"
                      />
                    </g>
                  </svg>
                </button>
                <button
                  id="download-qr-image-button-svg"
                  class="button"
                  @click="downloadQRImageAsSvg"
                  :aria-label="$t('Download QR Code as SVG')"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    viewBox="0 0 24 24"
                  >
                    <g
                      fill="none"
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                    >
                      <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                      <path
                        d="M5 12V5a2 2 0 0 1 2-2h7l5 5v4M4 20.25c0 .414.336.75.75.75H6a1 1 0 0 0 1-1v-1a1 1 0 0 0-1-1H5a1 1 0 0 1-1-1v-1a1 1 0 0 1 1-1h1.25a.75.75 0 0 1 .75.75m3-.75l2 6l2-6m6 0h-1a2 2 0 0 0-2 2v2a2 2 0 0 0 2 2h1v-3"
                      />
                    </g>
                  </svg>
                </button>
              </div>
            </div>
          </div>
        </div>
        <div id="settings" class="flex w-full grow flex-col items-start gap-8 text-start">
          <div>
            <label for="preset-selector">{{ $t('Preset') }}</label>
            <div class="flex flex-row items-center justify-center gap-2">
              <select
                id="preset-selector"
                class="secondary-button cursor-pointer text-start"
                :aria-label="$t('QR code preset')"
                v-model="selectedPreset"
              >
                <option v-for="(preset, index) in allPresets" :key="index" :value="preset">
                  {{ $t(preset.name) }}
                </option>
              </select>
              <button
                class="icon-button"
                @click="randomizeStyleSettings"
                :aria-label="$t('Randomize style')"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="40"
                  height="32"
                  viewBox="0 0 640 512"
                >
                  <path
                    fill="#888888"
                    d="M274.9 34.3c-28.1-28.1-73.7-28.1-101.8 0L34.3 173.1c-28.1 28.1-28.1 73.7 0 101.8l138.8 138.8c28.1 28.1 73.7 28.1 101.8 0l138.8-138.8c28.1-28.1 28.1-73.7 0-101.8L274.9 34.3zM200 224a24 24 0 1 1 48 0a24 24 0 1 1-48 0zM96 200a24 24 0 1 1 0 48a24 24 0 1 1 0-48zm128 176a24 24 0 1 1 0-48a24 24 0 1 1 0 48zm128-176a24 24 0 1 1 0 48a24 24 0 1 1 0-48zm-128-80a24 24 0 1 1 0-48a24 24 0 1 1 0 48zm96 328c0 35.3 28.7 64 64 64h192c35.3 0 64-28.7 64-64V256c0-35.3-28.7-64-64-64H461.7c11.6 36 3.1 77-25.4 105.5L320 413.8V448zm160-120a24 24 0 1 1 0 48a24 24 0 1 1 0-48z"
                  />
                </svg>
              </button>
            </div>
          </div>
          <div class="w-full">
            <label for="data">
              {{ $t('Data to encode') }}
            </label>
            <textarea
              name="data"
              class="text-input"
              id="url"
              rows="2"
              :placeholder="$t('data to encode e.g. a URL or a string')"
              v-model="data"
            />
          </div>
          <div class="w-full">
            <div class="mb-2 flex flex-row items-center gap-2">
              <label for="image-url">
                {{ $t('Logo image URL') }}
              </label>
              <button class="icon-button flex flex-row items-center" @click="uploadImage">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <g
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  >
                    <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                    <path
                      d="M17 21H7a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h7l5 5v11a2 2 0 0 1-2 2zm-5-10v6"
                    />
                    <path d="M9.5 13.5L12 11l2.5 2.5" />
                  </g>
                </svg>
                <p>{{ $t('Upload image') }}</p>
              </button>
            </div>
            <textarea
              name="image-url"
              class="text-input"
              id="url"
              rows="1"
              :placeholder="$t('Logo image URL')"
              v-model="image"
            />
          </div>
          <div class="w-full">
            <label for="image-margin">
              {{ $t('Image margin (px)') }}
            </label>
            <input
              class="text-input"
              id="image-margin"
              type="number"
              placeholder="0"
              v-model="imageMargin"
            />
          </div>
          <div class="flex w-full flex-row items-center gap-2">
            <label>{{ $t('Background color') }}</label>
            <input id="dotsColor" type="color" class="color-input" v-model="styleBackground" />
          </div>
          <div class="w-full">
            <label for="width">
              {{ $t('Width (px)') }}
            </label>
            <input
              class="text-input"
              id="width"
              type="number"
              placeholder="width in pixels"
              v-model="width"
            />
          </div>
          <div class="w-full">
            <label for="height">
              {{ $t('Height (px)') }}
            </label>
            <input
              class="text-input"
              id="height"
              type="number"
              placeholder="height in pixels"
              v-model="height"
            />
          </div>
          <div class="w-full">
            <label for="margin">
              {{ $t('Margin (px)') }}
            </label>
            <input class="text-input" id="margin" type="number" placeholder="0" v-model="margin" />
          </div>
          <div class="w-full">
            <label for="margin">
              {{ $t('Border radius (px)') }}
            </label>
            <input
              class="text-input"
              id="margin"
              type="number"
              placeholder="24"
              v-model="styleBorderRadius"
            />
          </div>
          <div class="flex w-full flex-row items-center gap-2">
            <label>{{ $t('Dots color') }}</label>
            <input id="dotsColor" type="color" class="color-input" v-model="dotsOptionsColor" />
          </div>
          <div class="w-full">
            <label>{{ $t('Dots type') }}</label>
            <div
              class="radiogroup"
              v-for="type in [
                'dots',
                'rounded',
                'classy',
                'classy-rounded',
                'square',
                'extra-rounded'
              ]"
              :key="type"
            >
              <input
                :id="'dotsOptionsType-' + type"
                type="radio"
                v-model="dotsOptionsType"
                :value="type"
              />
              <label :for="'dotsOptionsType-' + type">{{ $t(type) }}</label>
            </div>
          </div>

          <div class="flex w-full flex-row items-center gap-2">
            <label>{{ $t('Corners Square color') }}</label>
            <input
              id="cornersSquareColor"
              type="color"
              class="color-input"
              v-model="cornersSquareOptionsColor"
            />
          </div>
          <div class="w-full">
            <label>{{ $t('Corners Square type') }}</label>
            <div class="radiogroup" v-for="type in ['dot', 'square', 'extra-rounded']" :key="type">
              <input
                :id="'cornersSquareOptionsType-' + type"
                type="radio"
                v-model="cornersSquareOptionsType"
                :value="type"
              />
              <label :for="'cornersSquareOptionsType-' + type">{{ $t(type) }}</label>
            </div>
          </div>

          <div class="flex w-full flex-row items-center gap-2">
            <label>{{ $t('Corners Dot color') }}</label>
            <input
              id="cornersDotColor"
              type="color"
              class="color-input"
              v-model="cornersDotOptionsColor"
            />
          </div>
          <div class="w-full">
            <label>{{ $t('Corners Dot type') }}</label>
            <div class="radiogroup" role="radiogroup" v-for="type in ['dot', 'square']" :key="type">
              <input
                :id="'cornersDotOptionsType-' + type"
                type="radio"
                v-model="cornersDotOptionsType"
                :value="type"
              />
              <label :for="'cornersDotOptionsType-' + type">{{ $t(type) }}</label>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style lang="postcss" scoped>
label {
  @apply text-gray-700 dark:text-gray-100 text-lg font-semibold;
}

.text-input,
.secondary-button {
  @apply bg-zinc-50 dark:bg-zinc-800 text-zinc-900 dark:text-zinc-100 shadow hover:shadow-md transition-shadow rounded-lg;
}

.text-input,
input[type='color'],
input[type='radio'],
.button,
.secondary-button,
.icon-button {
  @apply outline-none focus-visible:ring-1 focus-visible:ring-zinc-700 dark:focus-visible:ring-zinc-200;
}

.text-input {
  @apply resize-none appearance-none ms-1 p-4 rounded w-full;
}

.color-input {
  @apply bg-transparent shadow p-0 border rounded box-border text-zinc-700 dark:text-zinc-100 focus-visible:shadow;
}

.vertical-border {
  @apply h-6 bg-slate-300 dark:bg-slate-700 w-1;
}

.button {
  @apply bg-zinc-100 dark:bg-zinc-700 text-zinc-900 dark:text-zinc-200 shadow-sm hover:shadow p-2 focus-visible:shadow-md rounded-lg;
}

.secondary-button {
  @apply outline-none p-1.5;
}

.icon-button {
  @apply outline-none hover:shadow rounded-sm;
}

input[type='radio'] {
  @apply m-3;
}

.radio-group {
  @apply flex flex-row items-center gap-1;
}

.radiogroup > label {
  @apply font-normal;
}
</style>
