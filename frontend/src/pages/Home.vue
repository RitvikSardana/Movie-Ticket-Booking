<template>
  <div class="p-8">
    <h1 class=" text-gray-900 font-bold text-[32px] pb-4">List of Movies</h1>

    <ul class="flex flex-col gap-y-2 cursor-pointer flex-wrap" >
      <router-link 
        :to = "{name:'Movies',params:{id:movie.name}}"
        v-for="movie in movies.data"

      >
          <Card
            :title="movie.title"
            :subtitle = "movie.synopsis"
            class="min-h-[180px] flex flex-col justify-around"
          >
        </Card>
      </router-link>
    </ul>
  </div>
</template>

<script setup>
import { Card, createListResource } from 'frappe-ui';

const movies = createListResource({
    doctype:"Movie",
    fields:['name','title','release_date','director','synopsis','poster'],
    auto:true
})


</script>

<style scoped>

</style>