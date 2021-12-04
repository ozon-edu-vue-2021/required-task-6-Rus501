<template>
  <Table
    :rows="rows"
    :total-pages="10"
    :current-page="currentPage"
    :static-paging="false"
    @getPage="infGetPage"
  >
    <TableColumn prop="id" title="ID" />
    <TableColumn prop="userId" title="Post ID" />
    <TableColumn prop="title" title="Post title" />
    <TableColumn prop="body" title="Post content" />
  </Table>
</template>

<script>
import Table from '@/components/table.vue'
import TableColumn from '@/components/table-column.vue'

import 'normalize.css'

export default {
  name: 'App',
  components: {
    Table,
    TableColumn
  },
  created() {
    this.blockingPromise = this.getPage(1)
  },
  data() {
    return {
      rows: [],
      currentPage: 1
    }
  },
  methods: {
    async getPage(number) {
      const request = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${number}`)
      this.rows = await request.json()
      this.currentPage = number
    },
    async infGetPage() {
      this.blockingPromise && await this.blockingPromise;
      const res = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${this.currentPage + 1}`);
      const newRows = await res.json();
      this.rows = [...this.rows, ...newRows];
      this.currentPage++;
    }
  },
}
</script>

<style>
body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
