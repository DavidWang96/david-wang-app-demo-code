<template>
  <div class="about-page">
    <div class="page-content">
      <div class="container">
        <h1
          v-char-busting="{ speed: 3000 }"
          class="page-title title-1 text-primary animate fade"
          v-text="$t('pages.about.title')"
        ></h1>

        <div class="about-page__bio">
          <div class="bio">
            <div class="row">
              <div class="col-xl-6 col-md-5">
                <div class="bio__img animate fade-right">
                  <div class="overflow">
                    <img
                      class="animate-mouse-move"
                      data-sensivity="60"
                      src="../assets/images/coding_wallpaper-1.jpg"
                      alt=""
                    />
                  </div>
                </div>
              </div>
              <div class="col-xl-6 col-md-7">
                <div class="bio__intro">
                  <h2
                    v-char-busting="{ speed: 4000 }"
                    class="text-primary animate fade-left"
                    v-text="$t('pages.about.bio_1_title')"
                  ></h2>
                  <p class="animate fade-left">
                    {{ $t('pages.about.bio_1_description') }}
                  </p>

                  <h3
                    v-char-busting="{ speed: 4000 }"
                    class="text-primary animate fade-left"
                    data-delay="100"
                    v-text="$t('pages.about.bio_2_title')"
                  ></h3>
                  <p class="animate fade-left" data-delay="100">
                    {{ $t('pages.about.bio_2_description') }}
                  </p>

                  <h3
                    v-char-busting="{ speed: 4000 }"
                    class="text-primary animate fade-left"
                    data-delay="200"
                    v-text="$t('pages.about.bio_3_title')"
                  ></h3>
                  <p class="animate fade-left" data-delay="200">
                    {{ $t('pages.about.bio_3_description') }}
                  </p>

                  <NuxtLink
                    class="main-page__button primary-button fw-bold animate short-fade-left"
                    data-delay="300"
                    to="/my-works"
                  >
                    <span class="fst-italic">{{
                      $t('pages.about.bio_my_works_title')
                    }}</span>
                    <i><IconUil:arrow-right /></i>
                  </NuxtLink>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="about-page__career">
          <div class="career">
            <div class="row">
              <div class="col-md-6">
                <div class="career__quote animate fade-right">
                  <p>
                    {{ $t('pages.about.career_quote') }}
                  </p>
                </div>

                <div class="career__additional animate fade-right">
                  <h3
                    v-char-busting="{ speed: 3000 }"
                    class="text-primary"
                    v-text="$t('pages.about.career_additional_title')"
                  ></h3>
                  <p>
                    {{ $t('pages.about.career_additional_description') }}
                  </p>
                </div>
              </div>
              <div class="col-md-6">
                <div class="career__specialization animate fade-left">
                  <div class="background">
                    <img
                      class="animate-scroll-motion"
                      data-sensivity="5"
                      src="../assets/images/coding_wallpaper-2.jpg"
                      alt=""
                    />
                  </div>
                  <p class="title animate fade">
                    {{ $t('pages.about.career_specialization_title') }}
                  </p>
                  <ul>
                    <li class="animate short-fade-right" data-delay="100">
                      <b>Front-end:</b> HTML, CSS, JavaScript (ES6+,
                      TypeScript), Vue.js
                    </li>
                    <li class="animate short-fade-right" data-delay="200">
                      <b>Back-end:</b> PHP, Laravel 9, Node.js
                    </li>
                    <li class="animate short-fade-right" data-delay="300">
                      <b>{{ $t('pages.about.career_specialization_1') }}:</b>
                      MySQL
                    </li>
                    <li class="animate short-fade-right" data-delay="400">
                      RESTful API, Docker, Git, WebSocket, Telegram Bot,
                      Electron
                      {{ $t('pages.about.career_specialization_2') }}
                    </li>
                  </ul>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="about-page__skills">
          <h3
            class="page-title title-2 text-primary animate fade"
            v-html="$t('pages.about.skills_title')"
          ></h3>

          <div v-if="isClient ? skills.length > 0 : true" class="skills">
            <div
              v-for="(item, index) in skills"
              :key="index"
              class="skill mouse-move-element"
            >
              <img
                :src="item.icon"
                class="animate fade"
                data-duration="1000"
                :data-delay="index * 30"
                alt=""
              />
            </div>
          </div>
        </div>
      </div>

      <div class="about-page__background">
        <img
          class="animate-scroll-motion"
          data-sensivity="3"
          src="../assets/images/coding_wallpaper-3.jpg"
          alt=""
        />
      </div>

      <div class="about-page__services">
        <div class="container">
          <h3
            class="page-title title-2 text-primary animate fade"
            v-html="$t('pages.about.services_title')"
          ></h3>

          <div v-if="isClient ? services.length > 0 : true" class="services">
            <div class="row">
              <div
                v-for="(service, index) in services"
                :key="index"
                class="col-md-6 animate short-zoom"
              >
                <div class="service" :data-delay="index * 100">
                  <div class="service__name">
                    <p>{{ service.name }}</p>
                  </div>
                  <div class="service__text" v-html="service.content"></div>
                  <div class="service__image">
                    <img :src="service.image" alt="" />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="about-page__my-works animate fade">
        <div class="container">
          <h3
            class="page-title title-2 text-primary animate fade"
            v-html="$t('pages.about.my_works_title')"
          ></h3>

          <p class="page-subtitle animate short-fade-up">
            {{ $t('pages.about.my_works_subtitle') }}
          </p>

          <div v-if="cases.length > 0" class="slider">
            <carousel
              ref="casesSlider"
              :items-to-show="3"
              :transition="500"
              :wrap-around="true"
            >
              <slide v-for="(slide, index) in cases" :key="index">
                <div class="slide" @click="openCaseModal(slide, index)">
                  <img :src="slide.preview" alt="" />
                </div>
              </slide>

              <template #addons>
                <navigation />
                <pagination />
              </template>
            </carousel>
          </div>
        </div>
      </div>
    </div>

    <content-modal
      v-if="selectedCase"
      ref="caseImageModalRef"
      class="case-preview-modal"
    >
      <template #header>
        <p class="case-preview-modal__title">{{ selectedCase.name }}</p>
        <p class="case-preview-modal__description">
          {{ selectedCase.short_text }}
        </p>
      </template>
      <template #content>
        <div class="case-preview-modal__content text-center">
          <img :src="selectedCase.preview" alt="Case Image" class="preview" />
        </div>
      </template>
    </content-modal>
  </div>
</template>

<script lang="ts" setup>
// Для слайдера в конце страницы - используем библиотеку vue3-carousel
import { Carousel, Slide, Pagination, Navigation } from 'vue3-carousel'
import { useAnimate } from '~/composables/useAnimate'
import { useAppLoader } from '~/stores/loader'

// Утилита перевода статичных текстов приложения
const { t, switchLangApiData } = useLang()

// Актуальный язык приложения
const locale = useState<string>('locale.setting').value

// Слушатель и создатель глобальных событий
const { $listen, $eventOff } = useNuxtApp()

// Среда выполнения кода
const isClient: boolean = process.client

// Применяем шаблон страницы к данному компоненту
definePageMeta({
  layout: 'page',
})

// Запускаем лоадер приложения
const appLoader = useAppLoader()
await appLoader.startLoading()

// Получаем ссылку на API приложения из .env
const apiHost: string =
  (import.meta.env.VITE_API_HOST as string) || 'http://example.com/api/'

// Создаем реактивные массивы для данных страницы
const skills = ref<Skill[]>([])
const services = ref<Service[]>([])
const cases = ref<Case[]>([])
const selectedCase = ref<Case | null>(null)

// Массив из элементов, наделенные визуальными mouse-move эффектами
const mouseMoveElements = ref<Array<HTMLElement>>([])

// Функция добавления mouse-move элементов с визуальными эффектами
const addMouseMoveElements = (): void => {
  const elements: NodeListOf<Element> = document.querySelectorAll(
    '.mouse-move-element'
  )
  mouseMoveElements.value = Array.from(elements) as Array<HTMLElement>
}

// Компонент модального окна
const caseImageModalRef = ref<Modal | null>(null)

// Компонент слайдера
const casesSlider = ref<Carousel | null>(null)

// Функция открытия модального окна и отображения данных элемента
const openCaseModal = async (caseItem: Case, index: number): Promise<void> => {
  selectedCase.value = caseItem
  await nextTick()

  if (caseImageModalRef.value) {
    caseImageModalRef.value.openModal()
  }

  if (casesSlider.value) {
    casesSlider.value.slideTo(index)
  }
}

// Создаем ограничение между выполнениями функции в целях экономии ресурсов устройства
let isThrottled: boolean = false

// Функция - обработчик события mouse-move
const handleMousemove = (event: MouseEvent): void => {
  if (!isThrottled) {
    isThrottled = true

    setTimeout(() => {
      isThrottled = false

      // Определяем текущее положение мыши
      const mouseX: number = event.clientX
      const mouseY: number = event.clientY

      // Суть данного визуального эффекта в том, чтобы применять CSS стили в зависимости от расстояния мыши до элемента
      mouseMoveElements.value.forEach((element) => {
        const elementRect: ClientRect = element.getBoundingClientRect()

        // Вычисляем координаты элементов
        const elementX: number = elementRect.left + elementRect.width / 2
        const elementY: number = elementRect.top + elementRect.height / 2

        // Вычисляем дистанцию между элементами и мышью
        const distance: number = Math.sqrt(
          Math.pow(mouseX - elementX, 2) + Math.pow(mouseY - elementY, 2)
        )

        // Создаем CSS свойства
        let grayscale: number
        if (distance <= 30) {
          grayscale = 0
        } else if (distance > 500) {
          grayscale = 100
        } else {
          grayscale = ((distance - 30) / 470) * 100
        }

        let opacity: number
        if (distance > 500) {
          opacity = 0.5
        } else {
          opacity = 1 - ((distance - 30) / 470) * 0.5
        }

        let scale: number
        if (distance <= 30) {
          scale = 1.2
        } else if (distance > 500) {
          scale = 1
        } else {
          scale = 1.2 - ((distance - 30) / 470) * 0.2
        }

        // Применяем CSS свойства
        element.style.filter = `grayscale(${grayscale}%)`
        element.style.opacity = `${opacity}`
        element.style.transform = `scale(${scale})`
      })
    }, 30)
  }
}

// Background - Фигуры для данной страницы
const figures: Figure[] = [
  { left: 14.713, top: 2.167, size: 0.92, figure: 'star', invert: true },
  { left: 6.463, top: 5.568, size: 0.78, figure: 'circle', invert: false },
  { left: 89.38, top: 4.295, size: 0.91, figure: 'cross', invert: false },
  { left: 70.15, top: 11.972, size: 0.71, figure: 'star', invert: false },
  { left: 11.718, top: 18.138, size: 0.79, figure: 'square', invert: true },
  { left: 6.568, top: 22.613, size: 0.96, figure: 'triangle', invert: false },
  { left: 91.8, top: 22.931, size: 0.89, figure: 'cross', invert: true },
  { left: 17.393, top: 31.782, size: 0.89, figure: 'circle', invert: false },
  { left: 81.1, top: 34.38, size: 0.79, figure: 'cross', invert: true },
  { left: 16.97, top: 50.79, size: 0.98, figure: 'circle', invert: true },
  { left: 83.34, top: 52.74, size: 0.83, figure: 'square', invert: false },
  { left: 7.146, top: 60.3, size: 0.77, figure: 'circle', invert: false },
  { left: 93.32, top: 63.74, size: 0.91, figure: 'cross', invert: true },
  { left: 46.505, top: 60.4, size: 0.98, figure: 'square', invert: true },
  { left: 18.33, top: 77.22, size: 0.7, figure: 'square', invert: false },
  { left: 89.75, top: 83.47, size: 0.8, figure: 'triangle', invert: true },
  { left: 5.412, top: 88.24, size: 0.9, figure: 'square', invert: false },
  { left: 83.39, top: 75.2, size: 0.86, figure: 'triangle', invert: true },
  { left: 91.8, top: 38.98, size: 0.7, figure: 'star', invert: false },
  { left: 5.675, top: 40.2, size: 0.77, figure: 'square', invert: false },
]

const { $event } = useNuxtApp()
$event('background-figures:init', figures)

// Загрузка всех необходимых API для страницы
try {
  const skillsData: Skill[] = await $fetch<Skill[]>(apiHost + 'api/skills')
  skills.value.push(...skillsData)
} catch (error) {
  console.error(error)
}

try {
  let servicesData: Service[] = await $fetch<Service[]>(
    apiHost + 'api/services'
  )
  servicesData = await switchLangApiData(locale, servicesData)
  services.value.push(...servicesData)
} catch (error) {
  console.error(error)
}

try {
  let casesData: Case[] = await $fetch<Case[]>(apiHost + 'api/cases')
  casesData = await switchLangApiData(locale, casesData)
  cases.value.push(...casesData)
} catch (error) {
  console.error(error)
}

// Обработчик глобального события смены языка
$listen('lang:changed', async (lang: string | unknown): Promise<void> => {
  if (typeof lang === 'string') {
    // При смене языка - меняем контент API данных на новых язык
    services.value = await switchLangApiData(lang, services.value)
    cases.value = await switchLangApiData(lang, cases.value)
  }
})

// Composable - хук, срабатывающий при обнаружении искомых HTML элементов
onDOMGenerated(
  async (): Promise<void> => {
    // Инициализируем mouse-move элементы и создаем прослушиватель события
    await addMouseMoveElements()
    window.addEventListener('mousemove', handleMousemove)

    // После срабатывания данного хука - следует считать что страница загружена
    await appLoader.stopLoading()
    await nextTick()

    // Активируем визуальные анимации HTML элементов
    useAnimate(true)
  },
  15000,
  ['.about-page__skills .skills', '.about-page__services .services']
)

onUnmounted((): void => {
  // После демонтирования компонента - снимаем прослушиватели событий мыши и смены языка
  if (isClient) {
    window.removeEventListener('mousemove', handleMousemove)
  }

  $eventOff('lang:changed')
})
</script>
