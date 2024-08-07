<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { Separator } from '@/components/ui/separator';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Search } from 'lucide-vue-next';
import { Input } from '@/components/ui/input';
import PlanetsTable from '@/components/PlanetsTable.vue';
import PlanetsGrid from '@/components/PlanetsGrid.vue';
import Loader from '@/components/Loader.vue';
import {
  Select,
  SelectContent,
  SelectGroup,
  SelectItem,
  SelectLabel,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';

const planets = ref([]);
const searchQuery = ref('');
const minPopulation = ref(0);
const maxPopulation = ref(Infinity);
const loading = ref(true);
const viewMode = ref('grid');
const currentPage = ref(1);
const itemsPerPage = ref(12);
const sortBy = ref('name');
const sortOrder = ref('asc');

const router = useRouter();
const route = useRoute();

const allPlanets = computed(() => planets.value);

watch(sortOrder, (newOrder) => {
  sortOrder.value = newOrder;
  updateUrl();
});

const fetchPlanets = async (url = 'https://swapi.dev/api/planets') => {
  loading.value = true;
  try {
    const response = await fetch(url);
    const data = await response.json();
    planets.value = [...planets.value, ...data.results];
    if (data.next) {
      await fetchPlanets(data.next);
    }
  } catch (error) {
    console.error('Error fetching planets:', error);
  } finally {
    loading.value = false;
  }
};

const formatDate = (dateString) => {
  const options = { year: 'numeric', month: 'long', day: 'numeric' };
  return new Date(dateString).toLocaleDateString(undefined, options);
};

const sortedPlanets = computed(() => {
  return planets.value.slice().sort((a, b) => {
    let result = 0;
    if (sortBy.value === 'name') {
      result = a.name.localeCompare(b.name);
    } else if (sortBy.value === 'created') {
      result = new Date(a.created) - new Date(b.created);
    }
    return sortOrder.value === 'asc' ? result : -result;
  });
});

const filteredPlanets = computed(() => {
  const effectiveMaxPopulation = (maxPopulation.value === '' || isNaN(maxPopulation.value))
    ? Infinity
    : Number(maxPopulation.value);

  return sortedPlanets.value.filter(planet => {
    const population = parseInt(planet.population, 10) || 0;
    return planet.name.toLowerCase().includes(searchQuery.value.toLowerCase()) &&
      population >= minPopulation.value &&
      population <= effectiveMaxPopulation;
  });
});

const paginatedPlanets = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return filteredPlanets.value.slice(start, end);
});

const pageCount = computed(() => {
  return Math.ceil(filteredPlanets.value.length / itemsPerPage.value);
});

watch(maxPopulation, (newValue) => {
  if (newValue === '') {
    maxPopulation.value = Infinity;
  }
});

watch(itemsPerPage, () => {
  currentPage.value = 1;
});

const updateItemsPerPage = (value) => {
  itemsPerPage.value = value;
};

const toggleSortOrder = () => {
  sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc';
};

const updateCurrentPage = (page) => {
  currentPage.value = page;
};

const updateUrl = () => {
  const query = {
    view: viewMode.value,
    ...(viewMode.value === 'grid' && { search: searchQuery.value }),
    ...(viewMode.value === 'grid' && { sort: sortOrder.value })
  };
  router.push({ query });
};

const handleViewChange = (mode) => {
  viewMode.value = mode;
  updateUrl();
};

watch(viewMode, () => {
  updateUrl();
});

watch(searchQuery, () => {
  if (viewMode.value === 'grid') {
    updateUrl();
  }
});

watch(() => route.query.view, (newView) => {
  viewMode.value = newView || 'grid';
  updateUrl();
});

watch(() => route.query.search, (newQuery) => {
  searchQuery.value = newQuery || '';
});

onMounted(() => {
  viewMode.value = route.query.view || 'grid';
  searchQuery.value = route.query.search || '';
  sortOrder.value = route.query.sort || 'asc';
  fetchPlanets();
});
</script>

<template>
  <div class="flex flex-col h-full p-4">
    <Tabs v-model="viewMode" class="w-full">
      <div class="flex items-center py-2">
        <h1 class="text-xl font-bold mr-6">SW Planets</h1>
        <TabsList class="ml-auto">
          <TabsTrigger value="grid" class="text-zinc-600 dark:text-zinc-200" @click="handleViewChange('grid')">
            Grid view
          </TabsTrigger>
          <TabsTrigger value="table" class="text-zinc-600 dark:text-zinc-200" @click="handleViewChange('table')">
            List view
          </TabsTrigger>
        </TabsList>
      </div>
      <Separator />
      <div v-if="viewMode === 'grid'"
        class="bg-background/95 py-4 backdrop-blur supports-[backdrop-filter]:bg-background/60 flex ">
        <form class="flex-1">
          <div class="relative">
            <Search class="absolute left-2 top-3 size-4 text-muted-foreground" />
            <Input v-model="searchQuery" placeholder="Search" class="pl-8" />
          </div>
        </form>
        <div class="flex-1 ml-10 flex items-center gap-4">
          <span>Sort</span>
          <Select v-model="sortOrder">
            <SelectTrigger>
              <SelectValue placeholder="Sort by Name" />
            </SelectTrigger>
            <SelectContent>
              <SelectGroup>
                <SelectLabel>Sort Order</SelectLabel>
                <SelectItem value="asc">Ascending</SelectItem>
                <SelectItem value="desc">Descending</SelectItem>
              </SelectGroup>
            </SelectContent>
          </Select>
        </div>
      </div>
      <TabsContent value="grid" class="m-0">
        <Loader v-if="loading" />
        <PlanetsGrid v-else :planets="filteredPlanets" :formatDate="formatDate" :currentPage="currentPage"
          :itemsPerPage="itemsPerPage" @updateCurrentPage="updateCurrentPage" />
      </TabsContent>

      <TabsContent value="table" class="m-0">
        <Loader v-if="loading" />
        <div v-if="allPlanets.length === 0 && !loading" class="flex items-center justify-center h-full py-4">
          <div class="text-gray-500 text-lg">No items found</div>
        </div>
        <PlanetsTable v-else :planets="allPlanets" :formatDate="formatDate" />
      </TabsContent>
    </Tabs>
  </div>
</template>