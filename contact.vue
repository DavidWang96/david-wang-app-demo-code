<template>
  <div class="contact-page">
    <div class="page-content">
      <div class="container">
        <h1
          v-char-busting="{ speed: 3000 }"
          class="page-title title-1 text-primary animate fade"
          v-text="$t('pages.contact.title')"
        ></h1>

        <div class="contact-form animate fade">
          <div class="contact-form__form animate fade-right">
            <p>
              <b>{{ $t('pages.contact.form_title') }}</b>
            </p>
            <p>
              {{ $t('pages.contact.form_descrioption') }}
            </p>
            <form @submit.prevent="sendMessage">
              <input
                v-model="name"
                required
                :class="sendErr && name === '' ? 'error' : ''"
                type="text"
                :placeholder="$t('pages.contact.form_enter_name')"
              />
              <textarea
                v-model="text"
                required
                :class="sendErr && text === '' ? 'error' : ''"
                type="text"
                :placeholder="$t('pages.contact.form_enter_message')"
              ></textarea>
              <button class="primary-button fw-bold">
                {{ $t('pages.contact.form_send_message') }}
                <i><IconUil:arrow-right /></i>
              </button>
            </form>
          </div>
          <div class="contact-form__chat animate fade-left">
            <div class="chat">
              <div v-if="messages.length === 0" class="empty">
                {{ $t('pages.contact.form_empty_messages') }}
              </div>

              <div
                v-for="(message, index) in messages"
                v-else
                :key="index"
                :class="[{ in: message.message_type === 'in' }, 'message']"
              >
                <strong>{{ message.username }}:</strong>
                {{ message.answer_text }}
              </div>
            </div>

            <div class="text-field">
              <input
                v-model="text"
                type="text"
                @keydown.enter="sendMessage"
              /><i @click="sendMessage"><IconUil:message /></i>
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

// Применяем шаблон страницы к данному компоненту
definePageMeta({
  layout: 'page',
})

// Интерфейс типизации сообщений чата
interface Message {
  username: string
  answer_text: string
  message_type: string
}

// Запускаем лоадер приложения
const appLoader = useAppLoader()
await appLoader.startLoading()

// Создаем реактивный массив, который будет хранить сообщения для чата
const messages = ref<Message[]>([])

// Текст, имя пользователя, простая валидация ошибок
const text = ref<string>('')
const name = ref<string>('')
const sendErr = ref<boolean>(false)

// Создаем переменную, хранящую WebSocket объект
let websocket!: WebSocket

// Получаем данные WebSocket-сервера из .env
const wsPort: number = +(import.meta.env.VITE_APP_WS_PORT || 8001)
const wsHost: string = (
  import.meta.env.VITE_APP_WS_HOST || 'localhost'
).toString()

// Функция отправки сообщений
const sendMessage = (): void | boolean => {
  // Проверяем заполнения необходимых полей name и text
  if (text.value.trim() === '' || name.value === '') {
    // Если одно из полей пустые - активируем ошибку и прерываем функцию
    sendErr.value = true
    return false
  }

  // Если данные заполнены - отправляем сообщение на WebSocket сервер
  sendErr.value = false
  websocket.send(
    JSON.stringify({ text: text.value.trim(), username: name.value })
  )

  // Также заносим сообщение в локальный массив сообщений
  messages.value.push({
    username: name.value.toString() || 'noname',
    answer_text: text.value.trim(),
    message_type: 'in',
  })

  text.value = ''
}

// WebSocket клиент доступен только в среде браузера
if (process.client) {
  websocket = new WebSocket(`wss://${wsHost}:${wsPort}`)
  websocket.addEventListener('message', (event) => {
    const message: Message = JSON.parse(event.data)
    messages.value.push(message)
  })
}

// Фигуры для фона данной страницы
const figures: Figure[] = [
  { left: 11.267, top: 7.429, size: 0.76, figure: 'circle', invert: false },
  { left: 81.6, top: 14.85, size: 0.84, figure: 'square', invert: false },
  { left: 92.66, top: 52.08, size: 0.79, figure: 'circle', invert: false },
  { left: 3.7064, top: 62.92, size: 0.7, figure: 'star', invert: false },
]

const { $event } = useNuxtApp()
$event('background-figures:init', figures)

onMounted(async (): Promise<void> => {
  // После монтирования компонента - останавливаем лоадер и запускаем анимацию HTML элементов
  await appLoader.stopLoading()

  useAnimate(true)
})

onUnmounted((): void => {
  // Закрываем WebSocket соединения при размонтировании компонента страницы
  websocket.close()
})
</script>
