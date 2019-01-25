<template lang='pug'>
  section
    <!-- <h1 class="header">R.Y.U.S.E.I</h1> -->
    <!-- <Search/> -->
    DragAndDrop
</template>

<script lang="ts">
import {
  Component,
  Vue
} from "nuxt-property-decorator"
import Rx from 'rxjs/Rx'
import VueRx from 'vue-rx'
import { State } from "vuex-class"
import Search from '~/components/Search.vue'
import DragAndDrop from '~/components/DragAndDrop.vue'
Vue.use(VueRx, Rx)

@Component({
  components: {
    Search,
    DragAndDrop
  }
})

export default class extends Vue {
  @State people
}
</script>
<style scoped>
.header {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana,
    sans-serif;
}

.cards {
  display: flex;
  flex-wrap: wrap;
}
</style>
