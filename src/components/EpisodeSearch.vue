<template>
  <div class="responsive-width">
    <div class="p-3">
      <h1 class="text-white">Search episodes in <span class="italic">{{ podcast?.meta?.title }}</span></h1>
      <input 
        class="p-1 w-full mt-1"
        v-model="search"
      />
      <ul>
        <li class="text-white mt-3" v-for="sr in searchResults" :key="sr._id">
          <div class="bg-gray-700 p-3">
            <h2>{{ sr.title }}</h2>
            <p class="text-xs text-gray-300">{{ prepareDescriptionString(sr.description) }}</p>
            <div class="text-sm mt-2">
              <font-awesome-icon icon="clock" /> {{ humanFriendlyDuration(sr.duration) }}
            </div>
          </div>
          <button 
            v-if="queue.map(qe => qe._id).includes(sr._id)"
            class="bg-red-500 w-full text-white p-1"
            @click="removeFromQueue(sr._id)"
          >Remove from queue</button>
          <button
            v-else
            class="bg-green-500 w-full text-white p-1"
            @click="addToQueue(sr._id)"
          >Add to queue</button>
        </li>

      </ul>
    </div>
  </div>
</template>

<script setup>
  import { cleanHTMLString, truncateString, humanFriendlyDuration } from '../Helpers'
  import { Shared } from '../State'
  import _ from 'lodash'
  import { ref, computed, watch } from 'vue'
  import { useStore } from 'vuex'
  import { useRoute } from 'vue-router'

  const store = useStore()
  const route = useRoute()

  const podcast = ref({})
  const search = ref('')
  const episodes = ref([])
  const searchResults = ref([])

  const queue = computed(() => store.state.queue)

  const getPodcasts = Shared.dexieDB.podcasts
    .where({ _id: route.params.id })
    .first()
    .then(pc => {
      podcast.value = pc
    })

  const getEpisodes = Shared.dexieDB.episodes
    .where({ podcast_id: route.params.id })
    .toArray()
    .then(eps => {
      episodes.value = eps
    })

  Promise.all([
    getPodcasts,
    getEpisodes
  ])

  const prepareDescriptionString = (string) => {
    if (string) {
      let parsedString = cleanHTMLString(string)
      return truncateString(parsedString, 150)
    } else {
      return 'No description available.'
    }
  }

  const addToQueue = (id) => {
    store.dispatch('addEpisodeToQueue', id)
  }

  const removeFromQueue = (id) => {
    store.dispatch('removeEpisodeFromQueue', id)
  }

  watch(search, _.debounce(function(value) {
    searchResults.value = episodes.value.filter(ep => {
      let lowerCaseDesc = ep.description.toLowerCase()
      let lowerCaseTitle = ep.title.toLowerCase()
      return lowerCaseDesc.includes(value.toLowerCase()) || lowerCaseTitle.includes(value.toLowerCase())
    })
  }, 1000))
</script>