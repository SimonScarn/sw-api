<script setup>
import { computed, watch } from 'vue';
import { Pagination, PaginationEllipsis, PaginationFirst, PaginationLast, PaginationList, PaginationListItem, PaginationNext, PaginationPrev } from '@/components/ui/pagination';
import { Button } from '@/components/ui/button';

const props = defineProps({
  currentPage: Number,
  pageCount: Number,
  updateCurrentPage: Function
});

const paginationItems = computed(() => {
  const items = [];
  const siblingCount = 1;
  const totalPages = props.pageCount;

  if (totalPages <= 1) return [];

  items.push({ type: 'page', value: 1 });

  if (props.currentPage > 3 + siblingCount) {
    items.push({ type: 'ellipsis', index: 1 });
  }

  for (let i = Math.max(2, props.currentPage - siblingCount); i <= Math.min(totalPages - 1, props.currentPage + siblingCount); i++) {
    items.push({ type: 'page', value: i });
  }

  if (props.currentPage < totalPages - 2 - siblingCount) {
    items.push({ type: 'ellipsis', index: 2 });
  }

  if (totalPages > 1) {
    items.push({ type: 'page', value: totalPages });
  }

  return items;
});

function handlePageChange(newPage) {
  if (newPage < 1 || newPage > props.pageCount) return;
  props.updateCurrentPage(newPage);
}

watch(() => props.currentPage, (newPage) => {
  if (newPage !== props.currentPage) {
    props.updateCurrentPage(newPage);
  }
});
</script>



<template>
  <Pagination
    v-if="pageCount > 1"
    v-slot="{ page }"
    :total="pageCount"
    :sibling-count="1"
    show-edges
    :default-page="currentPage"
    @update:page="handlePageChange"
  >
    <PaginationList v-slot="{ items }" class="flex items-center gap-1 justify-end">
      <PaginationFirst 
        @click="handlePageChange(1)" 
        :disabled="currentPage <= 1"
      />
      <PaginationPrev 
        @click="handlePageChange(currentPage - 1)"
        :disabled="currentPage <= 1"
      />
      
      <template v-for="(item, index) in paginationItems" :key="index">
        <PaginationListItem v-if="item.type === 'page'" :value="item.value" as-child>
          <Button
            class="w-10 h-10 p-0"
            :variant="item.value === currentPage ? 'default' : 'outline'"
            @click="handlePageChange(item.value)"
          >
            {{ item.value }}
          </Button>
        </PaginationListItem>
        <PaginationEllipsis v-else :key="item.type" :index="index" />
      </template>
      
      <PaginationNext 
        :disabled="currentPage >= pageCount"
        @click="handlePageChange(currentPage + 1)"
      />
      <PaginationLast 
        :disabled="currentPage >= pageCount"
        @click="handlePageChange(pageCount)"
      />
    </PaginationList>
  </Pagination>
</template>

