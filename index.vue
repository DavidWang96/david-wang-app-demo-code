<template>
  <div class="main-page">
    <div class="page-content d-flex flex-column vh-100">
      <div class="main-page__welcome flex-grow-1">
        <div
          v-if="isClient"
          class="main-page__3d-items d-flex justify-center align-items-center"
        >
          <div class="animate short-zoom w-100 h-100">
            <ItemsCloud />
          </div>
        </div>
        <div class="d-flex h-100 align-items-center">
          <div class="container">
            <div class="main-page__content position-relative">
              <h1
                v-char-busting="{ speed: 1500, blinkSpeed: 400 }"
                class="main-page__title text-primary animate short-fade-right"
                data-delay="0"
              >
                David Wang
              </h1>
              <p
                v-uppercase
                v-char-busting="{ speed: 2000, blinkSpeed: 1400 }"
                class="main-page__description fst-italic fs-4 animate short-fade-right"
                data-delay="200"
              >
                Web-developer
              </p>
              <NuxtLink
                class="main-page__button primary-button fw-bold animate short-fade-right"
                data-delay="400"
                to="/about"
              >
                <span class="fst-italic">{{ $t('pages.index.about') }}</span>
                <i><IconUil:arrow-right /></i>
              </NuxtLink>

              <div v-if="isClient" class="main-page__code-examples">
                <div class="code-examples animate fade">
                  <CodeExample
                    v-for="(example, index) in codeData.codeExamples"
                    :key="index"
                    :example="example"
                    :typing-speed="codeData.typingSpeed"
                  />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
// Composable для анимирования элементов
import { useAnimate } from '~/composables/useAnimate'

// Лоадер приложения
import { useAppLoader } from '~/stores/loader'

// Утилита перевода статичных текстов приложения
const { t } = useLang()
const appLoader = useAppLoader()

// Определение среды выполнения(клиент или сервер Node)
// Это нужно для того чтобы отрисовать или пропустить элементы, необходимые только для визуальных эффектов
const isClient = ref<boolean>(process.client)

// Применяем данный компонент к Layout страницы(page)
definePageMeta({
  layout: 'page',
})

// Интерфейс для типизации элементов визуального эффекта "набор кода HTML"
interface CodeData {
  codeExamples: CodeExample[]
  typingSpeed: number
}

// Начинаем загрузку страницы при входе в компонент
await appLoader.startLoading()

// У каждой страницы уникальные визуальные эффекты фигур, определяем массив для передачи данных
// в компонент Figures
const figures: Figure[] = [
  { left: 14, top: 7, figure: 'square', size: 0.6, invert: true },
  { left: 8, top: 17, figure: 'circle', size: 0.9, invert: false },
  { left: 40, top: 18, figure: 'cross', size: 0.7, invert: false },
  { left: 90, top: 10, figure: 'star', size: 0.9, invert: true },
  { left: 5, top: 38, figure: 'triangle', size: 0.5, invert: false },
  { left: 45, top: 45, figure: 'circle', size: 0.6, invert: true },
]

const { $event } = useNuxtApp()

// После объявления фигур - создаем событие background-figures:init и передаем массив
// в котором оно будет слушаться в компоненте Figures
$event('background-figures:init', figures)

// Создаем массив данных для визуального эффекта "набор кода HTML" на главной странице
const codeData: CodeData = {
  // Массив из текстовых элементов, содержат текст, отступы по X, Y и таймаут начала ввода текста(в секундах)
  codeExamples: [
    { text: '<html>', offset: { x: -50, y: -240 }, startAt: 2 },
    { text: '</html>', offset: { x: -50, y: 280 }, startAt: 4 },
    { text: '<head>', offset: { x: -20, y: -200 }, startAt: 6 },
    {
      text: '<title> David Wang - Web developer </title>',
      offset: { x: 0, y: -160 },
      startAt: 8,
    },
    { text: '</head>', offset: { x: -20, y: -120 }, startAt: 14 },
    { text: '<body>', offset: { x: -20, y: -30 }, startAt: 16 },
    { text: '</body>', offset: { x: -20, y: 230 }, startAt: 18 },
    { text: '<h1>', offset: { x: 0, y: 13 }, startAt: 20 },
    { text: '</h1>', offset: { x: 560, y: 13 }, startAt: 21 },
    { text: '<p>', offset: { x: 0, y: 80 }, startAt: 22 },
    { text: '</p>', offset: { x: 310, y: 80 }, startAt: 23 },
    { text: '<button>', offset: { x: 0, y: 162 }, startAt: 24 },
    { text: '</button>', offset: { x: 230, y: 162 }, startAt: 25 },
  ],

  // Скорость набора текста
  typingSpeed: 100,
}

onMounted((): void => {
  const { $listen, $eventOff } = useNuxtApp()

  // Создаем прослушивание события загрузки визуальных элементов облака - на главной странице
  $eventOff('items-cloud:onload')
  $listen('items-cloud:onload', async () => {
    // После того как данные загрузились - останавливаем лоадер приложения
    await appLoader.stopLoading()

    // После того как лоадер завершился - запускаем визуальные эффекты страницы
    useAnimate(true)
  })
})
</script>
