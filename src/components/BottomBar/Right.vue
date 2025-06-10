<template>
    <div class="right-group">
        <LyricsButton />
        <Volume />
        <button
            class="repeat"
            :class="{ 'repeat-disabled': settings.repeat == 'none' }"
            :title="settings.repeat == 'all' ? 'Repeat all' : settings.repeat == 'one' ? 'Repeat one' : 'No repeat'"
            @click="settings.toggleRepeatMode"
        >
            <RepeatOneSvg v-if="settings.repeat == 'one'" />
            <RepeatAllSvg v-else />
        </button>
        <button
            title="Toggle Shuffle Mode"
            :class="{ 'shuffle-enabled': isShuffleMode }"
            @click="toggleShuffleMode"
            class="shuffle-button"
        >
            <ShuffleSvg />
            <span v-if="isShuffleMode" class="shuffle-dot" />
        </button>
        <HeartSvg
            v-if="!hideHeart"
            title="Favorite"
            :state="queue.currenttrack?.is_favorite"
            @handleFav="() => $emit('handleFav')"
        />
    </div>
</template>

<script setup lang="ts">
import useQueue from '@/stores/queue'
import useSettings from '@/stores/settings'

import RepeatOneSvg from '@/assets/icons/repeat-one.svg'
import RepeatAllSvg from '@/assets/icons/repeat.svg'
import ShuffleSvg from '@/assets/icons/shuffle.svg'
import HeartSvg from '../shared/HeartSvg.vue'
import LyricsButton from '../shared/LyricsButton.vue'
import Volume from './Volume.vue'

const queue = useQueue()
const settings = useSettings()

import { ref, onMounted } from 'vue'

const isShuffleMode = ref(false)

onMounted(() => {
  fetch('/notsettings')
    .then(res => res.json())
    .then(config => {
      isShuffleMode.value = config.shuffleModeEnabled
    })
})

function toggleShuffleMode() {
  const newValue = !isShuffleMode.value
  isShuffleMode.value = newValue

  fetch('/notsettings/update', {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      key: 'shuffleModeEnabled',
      value: newValue,
    }),
  }).catch(err => {
    console.error('Failed to update shuffle mode:', err)
  })
}

defineProps<{
    hideHeart?: boolean
}>()

defineEmits<{
    (event: 'handleFav'): void
}>()
</script>

<style lang="scss">
.right-group {
    display: grid;
    justify-content: flex-end;
    grid-template-columns: repeat(5, max-content);
    align-items: center;
    height: 4rem;

    @include allPhones {
        width: max-content;
        height: unset;
    }

    button {
        height: 3rem !important;
        width: 3rem !important;
        background-color: transparent;
        border: solid 1px transparent;

        &:hover {
            border: solid 1px $gray3 !important;
            background-color: $gray !important;
        }
    }

    .lyrics,
    .repeat {
        svg {
            transform: scale(0.75);
        }

        &:active > svg {
            transform: scale(0.6);
        }
    }

    button.repeat.repeat-disabled {
        svg {
            opacity: 0.25;
        }
    }

    .heart-button {
        border: solid 1px $gray4 !important;
    }
    
    .shuffle-button {
    position: relative;
    }

    .shuffle-dot {
    position: absolute;
    top: 6px;
    right: 6px;
    width: 5px;
    height: 5px;
    border-radius: 50%;
    background-color: #ffffff;
    box-shadow: 0 0 2px #ffffff;
    }
}
</style>
