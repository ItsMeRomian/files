<template>
  <Table />
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import Table from "./components/Table.vue";
@Options({
  components: {
    Table
  }
})
export default class App extends Vue {}
</script>
