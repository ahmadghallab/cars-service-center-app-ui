<template>
<div>
  <v-tabs
    fixed-tabs
    show-arrows
    class="mb-3"
  >
    <v-tab>
      New Item
    </v-tab>
    <v-tab>
      Change Item
    </v-tab>
    <v-tab>
      New Outcome
    </v-tab>
    <v-tab>
      Change Outcome
    </v-tab>
    <v-tab>
      New Income
    </v-tab>
    <v-tab>
      Change Income
    </v-tab>
  </v-tabs>
  <NewItem v-if="newItem" />
</div>
</template>
<script>
import NewItem from '@/components/NewItem.vue'

export default {
  components: {
    NewItem
  },
  data: () => ({
    newItem: true,
  }),
  created () {

  }
}
</script>