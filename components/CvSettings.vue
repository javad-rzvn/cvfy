<script setup lang="ts">
import { SectionNameList } from '~/types/cvfy'
import { useCvState } from '~/data/useCvState'

const {
  formSettings,
  uploadCV,
  clearForm,
  resetForm,
} = useCvState()
const switchLocalePath = useSwitchLocalePath()
const i18n = useI18n()
const { downloadPdf } = usePrint()

const config = {
  layouts: ['one-column', 'two-column'],
  colors: [
    { name: 'pink', color: '#9D174D', darker: '#831843' },
    { name: 'purple', color: '#5B21B6', darker: '#4C1D95' },
    { name: 'blue', color: '#1E40AF', darker: '#1E3A8A' },
    { name: 'green', color: '#065F46', darker: '#064E3B' },
    { name: 'black', color: '#1F2937', darker: '#111827' },
  ],
  languages: [
    { name: 'es-name', code: 'es' },
    { name: 'en-name', code: 'en' },
    { name: 'id-name', code: 'id' },
    { name: 'fr-name', code: 'fr' },
    { name: 'zh-name', code: 'zh' },
    { name: 'de-name', code: 'de' },
    { name: 'ar-name', code: 'ar' },
    { name: 'pt-name', code: 'pt' },
    { name: 'fa-name', code: 'fa' },
  ],
}

// Watch for locale changes and update the page direction
// watch(
//   () => i18n.locale.value,
//   (newLocale) => {
//     const rtlLocales = ['fa', 'ar'] // Add all RTL locales here
//     const htmlElement = document.documentElement
//     const cvWrapper = document.querySelector('.cvWrapper');
//     if (rtlLocales.includes(newLocale)) {
//       cvWrapper.classList.add('rtl');
//     } else {
//       cvWrapper.classList.remove('rtl');
//     }
//   }
// )

watch(
  () => formSettings.value,
  (newValue, oldValue) => {
    localStorage.setItem(`cvSettings-${i18n.locale.value}`, JSON.stringify(newValue))
    if (newValue.activeColor !== oldValue.activeColor) {
      const newColor = getCurrentColor(newValue.activeColor)
      changeColor(newColor.color, newColor.darker)
    }
  },
  { deep: true },
)

const formSettingsHref = computed(() => {
  return `data:text/json;charset=utf-8,${encodeURIComponent(
    JSON.stringify({ formSettings: formSettings.value }),
  )}`
})

const availableLocales = computed(() => {
  return i18n.localeCodes.value.filter((locale: any) => !locale.includes('-'))
})

function changeColor(color: string, darker: string): void {
  formSettings.value.activeColor = color
  document.documentElement.style.setProperty('--primary', color)
  document.documentElement.style.setProperty('--primary-darker', darker)
}

function getCurrentColor(colorValue: string): {
  color: string
  darker: string
} {
  return (
    config.colors.find(color => color.color === colorValue)
    || config.colors[1]
  )
}
</script>

<template>
  <div class="settings">
    <div class="flex justify-between items-center title pt-2 px-6">
      <LandingLogo />
      <a
        class="buy-me-a-coffee"
        href="https://ko-fi.com/X8X4COWK0"
        target="_blank"
        rel="noopener noreferrer"
        aria-label="Buy me a coffee"
      >
        <img
          class="buy-me-a-coffee__image"
          src="https://cdn.ko-fi.com/cdn/kofi5.png?v=3"
          width="118px"
          height="30px"
          alt="Buy me a coffee button"
        >
      </a>
    </div>
    <h2 class="flex flex-wrap text-xl/normal pt-10 px-6 tracking-wide uppercase">
      <span class="title__text">
        {{ $t("cv-settings") }}
      </span>
    </h2>
    <form
      class="form mb-10"
      autocomplete="on"
    >
      <div class="form__section px-6 py-3">
        <button
          class="form__btn form__btn--ghost"
          type="button"
          @click="resetForm"
        >
          {{ $t("reset-settings") }}
        </button>
        <button
          class="form__btn form__btn--ghost"
          type="button"
          @click="clearForm"
        >
          {{ $t("clear-settings") }}
        </button>
      </div>

      <!-- LANGUAGE -->
      <fieldset class="form__section px-6 py-3">
        <legend class="form__legend">
          {{ $t("cv-language") }}
        </legend>
        <div class="flex flex-wrap gap-2 justify-start w-full">
          <nuxt-link
            v-for="locale in availableLocales"
            :key="locale"
            class="form__btn form__btn--ghost"
            :to="switchLocalePath(locale)"
            :exact="true"
          >
            {{ $t(`${locale}-name`) }}
          </nuxt-link>
        </div>
      </fieldset>
      <!-- LANGUAGE -->

      <!-- LAYOUT -->
      <fieldset class="form__section px-6 py-3">
        <legend class="form__legend">
          {{ $t("layout-theme") }}
        </legend>
        <div class="flex flex-wrap gap-2 justify-start">
          <label
            v-for="layout in config.layouts"
            :key="layout"
            tabindex="0"
            class="form__btn form__btn--ghost capitalize"
            :class="[
              {
                'form__btn--active':
                  layout === formSettings.layout,
              },
            ]"
          >
            {{ $t(layout) }}
            <input
              v-model="formSettings.layout"
              :value="layout"
              type="radio"
              class="sr-only"
            >
          </label>
        </div>
      </fieldset>
      <!-- LAYOUT -->

      <!-- COLOR THEME -->
      <fieldset class="form__section px-6 py-3">
        <legend class="form__legend">
          {{ $t("color-theme") }}
        </legend>
        <div class="flex flex-wrap gap-2 justify-start">
          <label
            v-for="color in config.colors"
            :key="color.color"
            tabindex="0"
            class="form__btn form__btn--color-theme capitalize"
            :class="[
              `form__btn--${color.name}`,
              {
                'form__btn--color-selected':
                  color.color === formSettings.activeColor,
              },
            ]"
            @keydown.enter="changeColor(color.color, color.darker)"
          >
            {{ $t(color.name) }}
            <input
              v-model="formSettings.activeColor"
              type="radio"
              class="sr-only"
              :value="color.color"
              @change="changeColor(color.color, color.darker)"
            >
          </label>
        </div>
      </fieldset>
      <!-- COLOR THEME -->

      <!-- PERSONAL DETAILS -->
      <fieldset class="form__section">
        <expansion-panel :panel-name="$t('personal-details')">
          <template #title>
            <legend class="form__legend">
              {{ $t("personal-details") }}
            </legend>
          </template>
          <template #content>
            <div class="grid grid-cols-2 gap-x-3 gap-y-10">
              <div class="form__group col-span-full">
                <span class="form__label">📷 {{ $t("profile-image") }} </span>
                <CvProfileImageUploader
                  v-model="formSettings.profileImageDataUri"
                />
              </div>
              <div class="form__group col-span-full">
                <label
                  class="form__label"
                  for="job-pos"
                >💼 {{ $t("job-title") }}</label>
                <input
                  id="job-pos"
                  v-model="formSettings.jobTitle"
                  class="form__control"
                  type="text"
                >
              </div>
              <div class="form__group">
                <label
                  class="form__label"
                  for="first-name"
                >👤 {{ $t("first-name") }}</label>
                <input
                  id="first-name"
                  v-model="formSettings.name"
                  class="form__control"
                  type="text"
                >
              </div>
              <div class="form__group">
                <label
                  class="form__label"
                  for="last-name"
                >👤 {{ $t("last-name") }}</label>
                <input
                  id="last-name"
                  v-model="formSettings.lastName"
                  class="form__control"
                  type="text"
                >
              </div>
              <div class="form__group col-span-full">
                <label
                  class="form__label"
                  for="email"
                >✉️ {{ $t("email") }}</label>
                <input
                  id="email"
                  v-model="formSettings.email"
                  class="form__control"
                  type="email"
                >
              </div>
              <div class="form__group">
                <label
                  class="form__label"
                  for="location"
                >📍 {{ $t("location") }}</label>
                <input
                  id="location"
                  v-model="formSettings.location"
                  class="form__control"
                  type="text"
                >
              </div>
              <div class="form__group">
                <label
                  class="form__label"
                  for="phone"
                >📱 {{ $t("phone-number") }}</label>
                <input
                  id="phone"
                  v-model="formSettings.phoneNumber"
                  class="form__control"
                  type="tel"
                >
              </div>
              <div class="form__group col-span-full">
                <label
                  class="form__label"
                  for="aboutme"
                >🌟 {{ $t("about-me") }}</label>
                <textarea
                  id="aboutme"
                  v-model="formSettings.aboutme"
                  class="form__control"
                  name="aboutme"
                  cols="30"
                  rows="10"
                />
              </div>
            </div>
          </template>
        </expansion-panel>
      </fieldset>
      <!-- PERSONAL DETAILS -->

      <!-- SKILLS -->
      <fieldset class="form__section grid gap-3">
        <expansion-panel :panel-name="$t('skills')">
          <template #title>
            <legend class="form__legend">
              {{ $t("skills") }}
            </legend>
          </template>
          <template #content>
            <div>
              <CvInputTags
                v-model="formSettings.jobSkills"
                tag-list-name="jobSkills"
                :tag-list-label="`🛠 ${$t('technical-skills')}`"
                :display="Boolean(formSettings.displayJobSkills)"
              />
              <CvInputTags
                v-model="formSettings.softSkills"
                tag-list-name="softSkills"
                :tag-list-label="`🧸 ${$t('soft-skills')}`"
                :display="Boolean(formSettings.displaySoftSkills)"
              />
              <CvInputTags
                v-model="formSettings.languages"
                tag-list-name="languages"
                :tag-list-label="`🌎 ${$t('languages')}`"
                :display="Boolean(formSettings.displayLanguages)"
              />
              <CvInputTags
                v-model="formSettings.interests"
                tag-list-name="interests"
                :tag-list-label="`🧸 ${$t('interests')}`"
                :display="Boolean(formSettings.displayInterests)"
              />
            </div>
          </template>
        </expansion-panel>
      </fieldset>
      <!-- SKILLS -->

      <!-- SOCIAL -->
      <fieldset class="form__section grid gap-3">
        <expansion-panel :panel-name="$t('social')">
          <template #title>
            <legend class="form__legend">
              {{ $t("social") }}
            </legend>
          </template>
          <template #content>
            <div>
              <CvDisplayCheckbox
                class="form__display-checkbox mb-10"
                :display-section="formSettings.displaySocial"
                section-name="social"
              />
              <div class="grid grid-cols-2 gap-x-3 gap-y-10">
                <div class="form__group col-span-full">
                  <label
                    class="form__label flex"
                    for="linkedin"
                  >
                    <svg class="form__icon rounded mr-1">
                      <use href="@/assets/sprite.svg#linkedin" />
                    </svg>
                    Linkedin
                  </label>
                  <input
                    id="linkedin"
                    v-model="formSettings.linkedin"
                    class="form__control"
                    type="text"
                  >
                </div>
                <div class="form__group col-span-full">
                  <label
                    class="form__label flex"
                    for="twitter"
                  >
                    <svg class="form__icon rounded mr-1">
                      <use href="@/assets/sprite.svg#twitter" />
                    </svg>
                    Twitter
                  </label>
                  <input
                    id="twitter"
                    v-model="formSettings.twitter"
                    class="form__control"
                    type="text"
                  >
                </div>
                <div class="form__group col-span-full">
                  <label
                    class="form__label flex"
                    for="github"
                  >
                    <svg class="form__icon mr-1">
                      <use href="@/assets/sprite.svg#github" />
                    </svg>
                    GitHub
                  </label>
                  <input
                    id="github"
                    v-model="formSettings.github"
                    class="form__control"
                    type="text"
                  >
                </div>
                <div class="form__group col-span-full">
                  <label
                    class="form__label flex"
                    for="website"
                  >
                    <svg class="form__icon mr-1">
                      <use href="@/assets/sprite.svg#website" />
                    </svg>
                    Website
                  </label>
                  <input
                    id="website"
                    v-model="formSettings.website"
                    class="form__control"
                    type="text"
                  >
                </div>
              </div>
            </div>
          </template>
        </expansion-panel>
      </fieldset>
      <!-- SOCIAL -->

      <!-- HISTORY SECTIONS -->
      <CvSettingsHistorySection
        v-for="(value, key) in SectionNameList"
        :key="key"
        :section="key"
        :name="value"
      />
      <!-- HISTORY SECTIONS -->

      <!-- CTA -->
      <div class="form__section flex flex-col p-6 gap-3">
        <button
          type="button"
          class="form__btn flex flex-col justify-center"
          @click="downloadPdf"
        >
          <span>{{ $t("download-cv-pdf") }}</span>
        </button>
        <label
          tabindex="0"
          class="form__btn flex justify-center"
        >
          {{ $t("upload-cv") }} (JSON)
          <input
            type="file"
            accept=".json"
            name="uploadCV"
            class="hidden"
            @change="uploadCV"
          >
        </label>
        <a
          :href="formSettingsHref"
          rel="noopener"
          :download="`CV_${formSettings.name}_${formSettings.lastName}_${$i18n.locale}.json`"
          class="form__btn flex justify-center"
        >{{ $t("download-cv-settings") }}
          (JSON)</a>
      </div>
      <!-- CTA -->
    </form>
  </div>
</template>

<style lang="postcss" scoped>
.settings {
  @apply bg-slate-50 bg-opacity-100 shadow-lg font-bold z-10;

  @media screen and (min-width: 1024px) {
    & {
      @apply overflow-y-auto;
    }
  }

  @media print {
    display: none;
    box-shadow: none;
    z-index: 0;
  }
}

.buy-me-a-coffee {
  flex-shrink: 1;

  &__image {
    width: 118px;
    height: 30px;
    font-size: 0.5rem;
  }
}
</style>
