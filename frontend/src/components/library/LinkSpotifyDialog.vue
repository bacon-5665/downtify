<template>
  <Teleport to="body">
    <Transition name="fade">
      <div
        v-if="playlist"
        class="fixed inset-0 z-[80] flex items-end justify-center bg-black/55 p-4 backdrop-blur-sm sm:items-center"
        @click.self="$emit('close')"
      >
        <div
          role="dialog"
          aria-modal="true"
          class="w-full max-w-md animate-rise rounded-panel border border-line-3 bg-surface p-6 shadow-float"
        >
          <h2 class="text-display text-lg font-semibold text-fg">
            {{ t('playlists.linkSpotifyTitle') }}
          </h2>
          <p class="mt-2 text-sm text-muted">
            {{ t('playlists.linkSpotifyBody') }}
          </p>
          <form
            :id="formId"
            class="mt-4 flex flex-col gap-4"
            @submit.prevent="save"
          >
            <input
              ref="urlInput"
              v-model="spotifyUrl"
              type="url"
              :placeholder="'https://open.spotify.com/playlist/...'"
              class="w-full rounded-control border border-line-2 bg-surface px-3 py-2 text-sm outline-none focus:border-accent"
              @keydown.enter="save"
            />
            <p
              v-if="error"
              class="flex items-start gap-2 text-sm text-danger"
              role="alert"
            >
              <AppIcon name="alert" :size="16" class="mt-0.5 shrink-0" />{{
                error
              }}
            </p>
          </form>
          <div
            class="mt-6 flex flex-col-reverse gap-2 sm:flex-row sm:justify-end"
          >
            <UiButton variant="ghost" type="button" @click="$emit('close')">
              {{ t('common.cancel') }}
            </UiButton>
            <UiButton
              variant="primary"
              type="submit"
              :form="formId"
              :loading="saving"
              :disabled="!spotifyUrl.trim()"
            >
              {{ t('common.link') }}
            </UiButton>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { computed, nextTick, ref, useId, watch as watchValue } from 'vue'
import AppIcon from '../ui/AppIcon.vue'
import UiButton from '../ui/UiButton.vue'
import API from '/src/model/api'
import { useI18n } from '/src/i18n'

const props = defineProps({
  playlist: { type: Object, default: null },
})
const emit = defineEmits(['close', 'linked'])

const { t } = useI18n()
const formId = `link-spotify-${useId()}`
const urlInput = ref(null)

const spotifyUrl = ref('')
const saving = ref(false)
const error = ref('')

watchValue(
  () => props.playlist,
  async (value) => {
    if (value) {
      spotifyUrl.value = ''
      error.value = ''
      await nextTick()
      urlInput.value?.focus()
    }
  }
)

const trimmedUrl = computed(() => spotifyUrl.value.trim())

async function save() {
  if (!trimmedUrl.value || saving.value) return
  saving.value = true
  error.value = ''
  try {
    await API.linkPlaylistToSpotify({
      playlist_name: props.playlist.name,
      spotify_url: trimmedUrl.value,
    })
    emit('linked')
    emit('close')
  } catch (err) {
    error.value = err?.response?.data?.detail || t('toast.actionFailed')
  } finally {
    saving.value = false
  }
}
</script>
