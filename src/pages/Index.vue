<template>
  <q-page class="row items-center justify-evenly">
    <!-- <p-1></p-1> -->
    <p-2></p-2>
  </q-page>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
import P1 from 'src/components/P1.vue';
import P2 from 'src/components/P2.vue';

@Component({
  components: { P1, P2 }
})
export default class PageIndex extends Vue {};
</script>
