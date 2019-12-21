<template>
  <v-app>
    <v-navigation-drawer
      v-model="drawer"
      absolute
      bottom
      temporary
    >
      <v-list
        nav
        dense
      >

        <!-- lg
        <v-list-group no-action>
          <template v-slot:activator>
            <v-list-item-content>
              <v-list-item-title>Customers</v-list-item-title>
            </v-list-item-content>
          </template>

          <v-list-item to="customers/create">
            <v-list-item-content>
              <v-list-item-title>Create New Customer</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-list-item to="customers/change">
            <v-list-item-content>
              <v-list-item-title>Change Existing Customers</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list-group>
        /lg -->

        <v-list-item-group
          v-model="group"
          active-class="deep-purple--text text--accent-4"
        >
          <v-list-item to="items">
            <v-list-item-title>Items</v-list-item-title>
          </v-list-item>

          <v-list-item to="makes">
            <v-list-item-title>Makes</v-list-item-title>
          </v-list-item>

          <v-list-item to="vehicles">
            <v-list-item-title>Vehicles</v-list-item-title>
          </v-list-item>

          <v-list-item to="customers">
            <v-list-item-title>Customers</v-list-item-title>
          </v-list-item>

          <v-list-item>
            <v-list-item-title>Finance</v-list-item-title>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar
      :clipped-left="clipped"
      fixed
      app
      dark
      color="amber"
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-toolbar-title v-text="title" />
      <!-- <v-spacer />
      <v-btn icon @click="$vuetify.theme.dark = !$vuetify.theme.dark">
        <v-icon v-if="$vuetify.theme.dark">mdi-brightness-4</v-icon>
        <v-icon v-else>mdi-brightness-7</v-icon>
      </v-btn> -->
    </v-app-bar>
    <v-content>
      <v-container>
        <nuxt />
      </v-container>
    </v-content>
    <!-- <v-footer
      :fixed="fixed"
      app
    >
      <span>&copy; Ahmad Ghallab</span>
    </v-footer> -->
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      title: 'Ghallab Portal'
    }
  }
}
</script>
<style>
  .v-pagination__navigation, .v-pagination__item, .v-pagination__item--active {
    box-shadow: none !important;
  }
  .v-pagination__item--active {
    pointer-events: none;
  }
  .v-pagination__navigation:focus, .v-pagination__item:focus {
    outline: none !important;
  }
</style>