<template>

    <div v-if="podcasts.length > 0" class="
        grid 
        grid-cols-3 
        sm:grid-cols-4 
        md:grid-cols-6 
        lg:grid-cols-8
        xl:grid-cols-10
        2xl:grid-cols-12
        auto-rows-max
      ">
        <StoredImage 
          :id="pc._id"
          :alt="pc.meta.title"
          :backupURL="pc.meta.imageURL"
          v-for="pc in podcasts"
          :key="pc._id"
          @click="router.push(`/podcasts/${pc._id}`)"
        />
    </div>

    <div v-else class="text-white h-full flex-1 flex justify-center items-center">
      <div class="flex flex-col">
        <button onclick="location.href = 'https://ltunes.gq';" class="mt-2 p-2 bg-green-500">Go to home page.</button>
      </div>
    </div>
</template>

<script setup>
  import { ref } from 'vue'
  import { useRouter } from 'vue-router'
  import { Shared } from '../State'
  import StoredImage from './StoredImage.vue'
  const router = useRouter()

  const podcasts = ref([])

  Shared.dexieDB.podcasts.toArray().then(result => {
    podcasts.value = result
  })
</script>
