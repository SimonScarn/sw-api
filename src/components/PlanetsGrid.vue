<template>
  <div>
    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
      <div v-if="loading">
        <Card class="animate-pulse mb-4 bg-grey-300">
          <CardHeader>
            <CardTitle class="bg-gray-200 h-6 w-3/4 mb-2"></CardTitle>
            <CardDescription class="bg-gray-200 h-4 w-1/2"></CardDescription>
          </CardHeader>
          <CardContent>
            <div class="bg-gray-200 h-4 w-full mb-2"></div>
            <div class="bg-gray-200 h-4 w-full mb-2"></div>
            <div class="bg-gray-200 h-4 w-full mb-2"></div>
          </CardContent>
        </Card>
      </div>
      <div v-for="planet in paginatedPlanets" :key="planet.url">
        <Card class="mb-4 h-full bg-[#E4E4E9]">
          <CardHeader>
            <CardTitle class="mb-2">{{ planet.name }}</CardTitle>
            <CardDescription>
              <strong>Population:</strong> {{ planet.population }}
            </CardDescription>
            <CardDescription>
              <strong>Rotation Period:</strong> {{ planet.rotation_period }}
            </CardDescription>
            <CardDescription>
              <strong>Gravity:</strong> {{ planet.gravity }}
            </CardDescription>
            <CardDescription>
              <strong>Created:</strong> {{ formatDate(planet.created) }}
            </CardDescription>
            <CardDescription>
              <strong>Url:</strong> {{ planet.url }}
            </CardDescription>
          </CardHeader>
          <CardFooter class="flex justify-end">
            <a :href="planet.url" target="_blank" class="inline-block">
              <Button variant="ghost">More Info</Button>
            </a>
          </CardFooter>
        </Card>
      </div>
    </div>
    <Separator class="mt-4" />
    <div class="mt-4">
      <div v-if="pageCount == 0" class="flex justify-center mt-8 text-gray-500">No items found</div>
      <PaginationComponent :currentPage="currentPage" :pageCount="pageCount" :updateCurrentPage="updateCurrentPage" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import PaginationComponent from '@/components/PaginationComponent.vue';
import Separator from './ui/separator/Separator.vue';

const props = defineProps({
  planets: Array,
  loading: Boolean,
  formatDate: Function,
});

const currentPage = ref(1);
const itemsPerPage = ref(12);

const paginatedPlanets = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return props.planets.slice(start, end);
});

const pageCount = computed(() => {
  return Math.ceil(props.planets.length / itemsPerPage.value);
});

const updateCurrentPage = (page) => {
  currentPage.value = page;
};

</script>
