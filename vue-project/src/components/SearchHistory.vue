<template>
  <div class="d-flex sidebar-shadow vh-100 overflow-scroll">
    <div class="text-center px-2">
      <nav
        aria-label="Search history pagination"
        class="mt-3"
        v-if="searchHistory.length"
      >
        <ul class="pagination justify-content-center">
          <li class="page-item" :class="{ disabled: currentPage === 1 }">
            <a class="page-link" href="#" @click.prevent="currentPage--"
              ><span aria-hidden="true">&laquo;</span></a
            >
          </li>
          <li
            class="page-item"
            v-for="page in totalPages"
            :key="page"
            :class="{ active: page === currentPage }"
          >
            <a class="page-link" href="#" @click.prevent="changePage(page)">{{
              page
            }}</a>
          </li>
          <li
            class="page-item"
            :class="{ disabled: currentPage === totalPages }"
          >
            <a class="page-link" href="#" @click.prevent="currentPage++"
              ><span aria-hidden="true">&raquo;</span></a
            >
          </li>
        </ul>
      </nav>
      <button
        v-if="anySelected"
        class="btn btn-danger my-2"
        @click.prevent="removeSelected"
      >
        Delete Selected
      </button>
      <div class="list-group">
        <a
          href="#"
          v-for="search in paginatedSearchHistory"
          :key="search.address"
          class="list-group-item list-group-item py-2 ms-1 ripple"
          @click="goToLocation(search.coords)"
        >
          <div class="d-flex">
            <input type="checkbox" v-model="search.selected" @click.stop />
            <div class="container mx-1">
              <div class="row mb-1">
                {{ search.address }}
              </div>
              <small class="row justify-content-center">{{
                search.localTime + " , " + search.timeZone
              }}</small>
            </div>
            <span class="close my-auto" @click.stop="removeSelected(search)"
              >&times;</span
            >
          </div>
        </a>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    searchHistory: Array,
    goToLocation: Function,
    clearMarkers: Function,
  },
  data() {
    return {
      currentPage: 1,
      itemsPerPage: 10,
      localSearchHistory: [...this.searchHistory],
    };
  },
  computed: {
    paginatedSearchHistory() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = this.currentPage * this.itemsPerPage;
      return this.searchHistory.slice(start, end);
    },
    totalPages() {
      return Math.ceil(this.searchHistory.length / this.itemsPerPage);
    },
    anySelected() {
      return this.searchHistory.some((search) => search.selected);
    },
  },
  methods: {
    changePage(page) {
      this.currentPage = page;
    },
    removeSelected(search) {
      this.localSearchHistory = this.searchHistory.filter(
        (s) => s.address !== search.address && !s.selected
      );

      if (this.paginatedSearchHistory.length === 0 && this.currentPage > 1) {
        this.currentPage--;
      }
      this.$emit("update:searchHistory", this.localSearchHistory);
    },
  },
  watch: {
    searchHistory(newVal) {
      this.localSearchHistory = [...newVal];
    },
  },
};
</script>

<style scoped>
.close {
  font-size: 20px;
  cursor: pointer;
}
</style>
