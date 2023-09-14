<template>
  <div class="my-cases-page">
    <div class="page-content">
      <div class="container">
        <h1
          v-char-busting="{ speed: 3000 }"
          class="page-title title-1 text-primary animate fade"
          v-text="$t('pages.my_works.title')"
        ></h1>

        <p class="page-subtitle animate short-fade-up">
          {{ $t('pages.my_works.subtitle') }}
        </p>

        <div class="my-cases-page__items animate fade">
          <template v-if="isClient">
            <div
              v-if="viewportLoaded"
              :class="`case-items ${inMotion ? 'in-motion' : ''}`"
            >
              <div
                v-for="(viewport, viewPortIndex) in viewPortCases"
                :key="viewPortIndex"
                class="cases-viewport d-flex flex-nowrap"
                :style="`
                ${
                  viewport.motionType === 'appear-left'
                    ? 'left: -' +
                      100 / casesX +
                      '%; transform: translateX(' +
                      100 / casesX +
                      '%);'
                    : ''
                }
                ${
                  viewport.motionType === 'appear-right'
                    ? 'transform: translateX(-' + 100 / casesX + '%);'
                    : ''
                }
            `"
                :class="`${viewport.motionType}`"
              >
                <div
                  v-for="(item, itemIndex) in viewport.cases"
                  :key="item.id"
                  :style="`
                ${
                  item.motionType
                    ? 'left:' + ((itemIndex - 1) * 100) / casesX + '%;'
                    : ''
                }
                flex: 1 0 ${100 / casesX}%;
                width: ${100 / casesX}%;
              `"
                  :class="`item ${item.motionType || ''}`"
                  @click="openCaseModal(item)"
                  @mousemove="
                    casesHoverIndexes = {
                      x: viewPortIndex + 1,
                      y: itemIndex + 1,
                    }
                  "
                >
                  <div class="item__content">
                    <div class="item__header">
                      {{ item.number.toString().padStart(2, '0') }}
                    </div>
                    <div class="item__body">
                      <h5 v-char-busting="{ speed: 5000 }" class="item__title">
                        {{ item.name }}
                      </h5>
                      <p class="item__text">{{ item.short_text }}</p>
                    </div>
                    <div
                      class="item__preview"
                      :style="`background-color: ${
                        themeSetting === 'dark'
                          ? item.preview_color_darker
                          : item.preview_color_darker
                      }`"
                    >
                      <img
                        :src="item.preview"
                        class="item__image"
                        alt="Case Image"
                      />
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </template>

          <template v-else>
            <div class="case-items">
              <div v-for="item in casesItems" :key="item.id" class="item">
                <div class="item__content">
                  <div class="item__header">
                    {{ item.number.toString().padStart(2, '0') }}
                  </div>
                  <div class="item__body">
                    <h5 v-char-busting="{ speed: 5000 }" class="item__title">
                      {{ item.name }}
                    </h5>
                    <p class="item__text">{{ item.short_text }}</p>
                  </div>
                  <div class="item__preview">
                    <img
                      :src="item.preview"
                      class="item__image"
                      alt="Item Image"
                    />
                  </div>
                </div>
              </div>
            </div>
          </template>
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
import { IThemeSettingOptions } from '~/utils/theme'
import { useAnimate } from '~/composables/useAnimate'
import { useAppLoader } from '~/stores/loader'

// Утилита для перевода статичных текстов приложения
const { t, switchLangApiData } = useLang()

// Метода прослушивания и объявления глобальных событий приложения
const { $listen, $eventOff } = useNuxtApp()

// Получаем актуальное значение языка приложения
const locale = useState<string>('locale.setting').value

// Лоадер приложения
const appLoader = useAppLoader()

// Получаем актуальное значение темы приложения
const themeSetting = useState<IThemeSettingOptions>('theme.setting')

// Определение среды выполнения(клиент или сервер Node)
const isClient = ref<boolean>(process.client)

// Флаг, определяющий готовность работы многомерного слайдера, для среды выполнения на Node - он не нужен
const viewportLoaded = ref<boolean>(!isClient.value)

// Применяем шаблон страницы к данному компоненту
definePageMeta({
  layout: 'page',
})

// Интерфейс типизации объекта наведения на элемент слайдера
interface HoverIndex {
  x: number
  y: number
}

// Интерфейс горизонтального ряда для слайдера
interface Viewport {
  motionType: string
  cases: Case[]
}

// Размерность элементов слайдера по вертикали и горизонтали
let casesX = 3
let casesY = 3

// Реактивный массив из элементов
const casesItems = ref<Case[]>([])

// Текущий выбранный элемент для отображения его данных в модальном окне
const selectedCase = ref<Case | null>(null)

// Получаем ссылку на API приложения из .env
const apiHost: string =
  (import.meta.env.VITE_API_HOST as string) || 'http://example.com/api/'

// Создаем асинхронную функцию для временных ожиданий, она пригодиться при пролистывания элементов слайдера
const delay = (ms: number): Promise<void> => {
  return new Promise(
    (resolve): ReturnType<typeof setTimeout> => setTimeout(resolve, ms)
  )
}

// Объявляем компонент модального окна, оно нужно для отображения данных выбранного элемента API
const caseImageModalRef = ref<Modal | null>(null)

// Функция открытия модального окна при выборе API элемента
const openCaseModal = async (caseItem: Case): Promise<void> => {
  selectedCase.value = caseItem
  await nextTick()

  // Если компонент модального окна прогружен, запускаем его внутренний метод openModal()
  if (caseImageModalRef.value) {
    caseImageModalRef.value.openModal()
  }
}

// Запускаем лоадер приложения
await appLoader.startLoading()

// Общий массив API элементов
let items: Case[] = []

// Массив дополнительных API элементов - он понадобится для пролистывания элементов слайдера
let additionalCases: Case[] = []

// Массив из горизонтальных рядов слайдера
const viewPortCases = ref<Viewport[]>([])

// Объект содержащий координаты наведенного элемента на слайдере
// Он нужен чтобы не перелистывать тот ряд по X и Y в котором имеется наведенный мышью - элемент
const casesHoverIndexes = ref<HoverIndex>({
  x: 0,
  y: 0,
})

// Функция генерация случайного числа заданного диапозона с заданными исключениями
const getRandomInt = (
  min: number,
  max: number,
  excludedNumber: number | null = null
): number => {
  let randomNum: number = Math.floor(Math.random() * (max - min + 1)) + min

  if (excludedNumber !== null) {
    while (randomNum === excludedNumber) {
      randomNum = Math.floor(Math.random() * (max - min + 1)) + min
    }
  }

  return randomNum
}

// Флаг, указывающий на момент движения слайдера, он нужен для динамических CSS стилей
// а также для предотвращения нежелательных операций со слайдером во время его пролистывания
const inMotion = ref<boolean>(false)

// Флаг указывающий на момент начала движения, он нужен для динамической прорисовки элементов и правильной анимации
const startMotion = ref<boolean>(false)

// Тип движения слайдера - необходим для динамических стилей
const motionType = ref<string>('back')

// Основная функция анимации слайдера
const casesMotionAnimate = async (): Promise<void> => {
  startMotion.value = true
  // Перед началом анимации сделаем небольшой интервал времени для прогрузки элементов слайдера
  await delay(200)
  inMotion.value = true
  await delay(500)

  // После переключения - сбрасываем в рядах слайдера значения движения
  viewPortCases.value.forEach((viewport: Viewport): void => {
    viewport.motionType = ''

    viewport.cases.forEach((el: Case): void => {
      el.motionType = ''
    })
  })

  // А также снимаем флажки - начала движения и самого движения
  startMotion.value = false
  inMotion.value = false
}

// Функция, оптимизирующая вертикальное движение слайдера, так как технически оно более сложное чем движение по горизонтали
const moveCasesVertical = (
  casesArray: Viewport[],
  columnIndex = 0,
  addToEnd = false,
  itemToAdd: Case
): Viewport[] => {
  const columnToAdd: Case[] = casesArray.map(
    (row: Viewport): Case => row.cases[columnIndex]
  )

  // Переменная в которой будем хранить удаленный элемент из слайдера для записи его в массив дополнительных элементов
  let removedElement: Case

  // Определяем сторону движения слайдера и в зависимости от стороны - удаляем элемент
  if (addToEnd) {
    removedElement = columnToAdd.shift()!
    columnToAdd.push(itemToAdd)
  } else {
    removedElement = columnToAdd.pop()!
    columnToAdd.unshift(itemToAdd)
  }

  for (let i = 0; i < casesArray.length; i++) {
    casesArray[i].cases[columnIndex] = columnToAdd[i]
  }

  // После всех операций над слайдером - добавляем убранный элемент из слайдера в additionalCases
  additionalCases.push(JSON.parse(JSON.stringify(removedElement)))

  // Возвращаем измененный массив элементов слайдера
  return casesArray
}

// Основная функция перелистывания слайдера
const addCaseToViewPort = async (
  el: Case,
  viewportNumber = 1,
  addingType = 'to-start',
  vertical = 0
): Promise<void> => {
  // Находим индекс добавляемого элемента на слайдер из additionalCases
  const index: number = additionalCases.findIndex(
    (item): boolean => item === el
  )

  // После добавления элемента - удаляем его из массива дополнительных элементов
  if (index !== -1) {
    additionalCases.splice(index, 1)
  }

  // Движение по вертикали или горизонтали определяется значением viewportNumber
  // если значение больше нуля - щначит движение будет по горизонтали, иначе - по вертикали
  if (viewportNumber > 0) {
    // addingType принимает 2 значения: to-start или to-end, в зависимости от значения - слайдер будет двигаться вперед или назад
    switch (addingType) {
      case 'to-start': {
        // Добавляем добавляемый элемент в ряд
        viewPortCases.value[viewportNumber - 1].cases = [
          el,
          ...viewPortCases.value[viewportNumber - 1].cases,
        ]

        // Задаем данному ряду тип движения - appear-left
        // Это присвоит соответствующий CSS стиль с учетом, что элемент появился слева
        viewPortCases.value[viewportNumber - 1].motionType = 'appear-left'

        // После добавления элемента - запускаем анимацию и слайдер начинает движение
        await casesMotionAnimate()

        // После того как слайдер завершил движение - обновляем массивы
        additionalCases.push(
          JSON.parse(
            JSON.stringify(
              viewPortCases.value[viewportNumber - 1].cases[
                viewPortCases.value[viewportNumber - 1].cases.length - 1
              ]
            )
          )
        )
        viewPortCases.value[viewportNumber - 1].cases.pop()

        // По аналогии - работают остальные виды движения
        break
      }

      case 'to-end': {
        viewPortCases.value[viewportNumber - 1].cases.push(el)

        viewPortCases.value[viewportNumber - 1].motionType = 'appear-right'
        await casesMotionAnimate()

        additionalCases.push(
          JSON.parse(
            JSON.stringify(viewPortCases.value[viewportNumber - 1].cases[0])
          )
        )
        viewPortCases.value[viewportNumber - 1].cases.splice(0, 1)

        break
      }
    }
  } else if (vertical > 0) {
    switch (addingType) {
      case 'to-start': {
        el.motionType = 'appear-top motion-down'
        let casesCopy: Viewport[] = JSON.parse(
          JSON.stringify(viewPortCases.value)
        )
        viewPortCases.value[0].cases.splice(vertical, 0, el)

        viewPortCases.value.forEach((viewport: Viewport): void => {
          viewport.cases[vertical - 1].motionType = 'motion-down'
        })

        await casesMotionAnimate()

        casesCopy = moveCasesVertical(casesCopy, vertical - 1, false, el)
        viewPortCases.value = casesCopy
        break
      }

      case 'to-end': {
        el.motionType = 'appear-bottom motion-up'
        let casesCopy: Viewport[] = JSON.parse(
          JSON.stringify(viewPortCases.value)
        )
        viewPortCases.value[viewPortCases.value.length - 1].cases.splice(
          vertical,
          0,
          el
        )

        viewPortCases.value.forEach((viewport: Viewport): void => {
          viewport.cases[vertical - 1].motionType = 'motion-up'
        })

        await casesMotionAnimate()

        casesCopy = moveCasesVertical(casesCopy, vertical - 1, true, el)
        viewPortCases.value = casesCopy
        break
      }
    }
  }
}

// Функция случайного перелистывания слайдера
const changeCase = async (): Promise<void> => {
  // Определяем данные из всевозможных вариантов учитывая также - элемент, который наведен мышью
  // Это нужно для того, чтобы наведенный элемент не перелистывался
  const viewPortNumber: number =
    Math.random() < 0.5 || casesX === 1
      ? getRandomInt(1, casesY, casesHoverIndexes.value.x)
      : 0
  const addingType: string = Math.random() < 0.5 ? 'to-start' : 'to-end'
  const vertical: number =
    viewPortNumber === 0
      ? getRandomInt(1, casesX, casesHoverIndexes.value.y)
      : 0

  // После определения данных - запускаем движение слайдера
  await addCaseToViewPort(
    additionalCases[0],
    viewPortNumber,
    addingType,
    vertical
  )
}

// Функция обнавления API данных при переключении языка приложения
const changeCasesLang = async (lang: string): Promise<void> => {
  // Для верной прорисовки элементов - снимаем флаг viewportLoaded
  viewportLoaded.value = false

  // Вызываем универсальный метод из Composable для перевода данных из API на новый язык
  items = await switchLangApiData(lang, items)

  // Тоже самое делаем с массивом дополнительных элементов
  additionalCases = await switchLangApiData(lang, additionalCases)

  // Затем проходимся по элементам всех рядов и также меняем язык у этих элементов
  viewPortCases.value.map(async (viewPort): Promise<Viewport> => {
    viewPort.cases = await switchLangApiData(
      lang,
      JSON.parse(JSON.stringify(viewPort.cases))
    )
    return viewPort
  })

  // После обнавления данных - активируем viewportLoaded для прорисовки новых данных в компоненте
  viewportLoaded.value = true
}

// Функция для автоматического определения размерности слайдера
const setCasesSize = (): void => {
  // Для того чтобы не обновлять данные на те же самые - создаем флаг
  let isSizeChanged = false

  // В зависимости от ширины окна браузера - определяем размерность слайдера
  if (window.innerWidth <= 992 && window.innerWidth >= 460) {
    if (casesY !== 3 || casesX !== 2) {
      isSizeChanged = true
      casesY = 3
      casesX = 2
    }
  } else if (window.innerWidth <= 460) {
    if (casesY !== 3 || casesX !== 1) {
      isSizeChanged = true
      casesY = 3
      casesX = 1
    }
  } else if (casesY !== 3 || casesX !== 3) {
    isSizeChanged = true
    casesY = 3
    casesX = 3
  }

  // Если размерность слайдера был обновлен - переинициализируем слайдер и его данные
  if (isSizeChanged || !viewportLoaded.value) {
    if (startMotion.value) {
      setTimeout((): void => {
        viewportLoaded.value = false
        initCasesData()
        viewportLoaded.value = true
      }, 750)
    } else {
      viewportLoaded.value = false
      initCasesData()
      viewportLoaded.value = true
    }
  }
}

// Функция - обработчик события visibilitychange
const handleVisibilityChange = (): void => {
  // Если по каким то причинам - владка стала неактивной - то останавливаем интервальное переключение слайдера во избежания багов
  if (document.hidden) {
    if (addingCasesInterval !== null) {
      clearInterval(addingCasesInterval)
    }
  } else {
    addingCasesInterval = setInterval(async (): Promise<void> => {
      await changeCase()
    }, 3000)
  }
}

// Функция - инициализация API данных для слайдера
const initCasesData = (): void => {
  // Присваиваем первичные данные для элементов
  for (let i = 0; i < viewPortCases.value.length; i++) {
    viewPortCases.value[i] = {
      motionType: '',
      cases: [],
    }
  }

  additionalCases = []

  // Переводим API элементы на актуальный язык приложения
  items = switchLangApiData(locale, items)

  // Расставляем API элементы на слайдер в соответствии с её размерностью
  items.forEach((item: Case, index: number): void => {
    if (index < casesX * casesY) {
      if (!viewPortCases.value[Math.floor(index / casesX)]) {
        viewPortCases.value[Math.floor(index / casesX)] = {
          motionType: '',
          cases: [],
        }
      }
      viewPortCases.value[Math.floor(index / casesX)].cases.push(item)
    } else {
      // Если элементов больше чем их можно разместить на слайдер - то заполняем лишними элементами массив additionalCases
      // Он будет необходим для показа новых элементов слайдера
      additionalCases.push(item)
    }
  })
}

// Создаем переменную, хранящую интервал между переключениями слайдера
let addingCasesInterval: ReturnType<typeof setInterval> | null = null

// Обработчик глобального события изменения языка приложения
$listen('lang:changed', async (lang: string | unknown): Promise<void> => {
  if (typeof lang === 'string') {
    // Переводим все API данные на новый доступный язык
    await changeCasesLang(lang || 'ru')
  }
})

// Создаем визуальные эффекты фигур для страницы
const figures: Figure[] = [
  { left: 6, top: 10, figure: 'star', size: 0.6, invert: true },
  { left: 90, top: 38, figure: 'circle', size: 0.9, invert: false },
  { left: 10, top: 18, figure: 'cross', size: 0.7, invert: false },
]

const { $event } = useNuxtApp()

// После создания фигур - запускаем событие и отправляем данные
$event('background-figures:init', figures)

// Загрузка данных для страницы через API Back-end
try {
  // Данный метод получения данных работает как в среде клиента, так и в среде Node
  const casesData: Case[] = await $fetch<Case[]>(apiHost + 'api/cases')
  items.push(...casesData)

  items = switchLangApiData(locale, items)

  casesItems.value.push(...items)
} catch (error) {
  console.error(error)
}

onMounted(async (): Promise<void> => {
  // Если среда выполнения - браузер, то создаем слайдер
  // Если среда - Node, то пробрасываем полученные данные без создания слайдера
  if (process.client) {
    // Определяем размерность слайдера
    setCasesSize()

    // Слушаем события изменения размера окна и видимости
    window.addEventListener('resize', setCasesSize)
    document.addEventListener('visibilitychange', handleVisibilityChange)

    viewportLoaded.value = true

    // Ожидаем полную прорисовку HTML элементов
    await nextTick()
    await delay(50)

    // Присваиваем классы для задания анимации элементов
    document
      .querySelectorAll<HTMLElement>('.case-items .item')
      .forEach((item: HTMLElement, index: number): void => {
        item.classList.add('animate')
        item.classList.add('short-zoom')
        item.setAttribute('data-duration', (1000).toString())
        item.setAttribute('data-delay', (index * 100).toString())
      })

    // После всех этих действий - останавливаем лоадер приложения
    await appLoader.stopLoading()

    // Запускаем анимации HTML элементов
    useAnimate(true)

    // Создаем интервал переключения слайдера каждые 3 секунды
    addingCasesInterval = setInterval(async (): Promise<void> => {
      await changeCase()
    }, 3000)
  }
})

onUnmounted((): void => {
  // При демонтирования компонента - отключаем прослуживание всех используемых событий
  if (process.client) {
    if (addingCasesInterval !== null) {
      clearInterval(addingCasesInterval)
    }

    window.removeEventListener('resize', setCasesSize)
    document.removeEventListener('visibilitychange', handleVisibilityChange)

    $eventOff('lang:changed')
  }
})
</script>
