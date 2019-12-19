<template>
 <v-card
    outlined
  >
    <v-form>
        <v-container>
            <v-row>
                <v-col cols="12" sm="6" md="3">
                    <v-text-field
                        label="Item Name"
                    ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6" md="3">
                    <v-text-field
                        label="Item Code"
                    ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6" md="3">
                    <v-select
                    :items="units"
                    label="Unit"
                    ></v-select>
                </v-col>
                <v-col cols="12" sm="6" md="3">
                    <v-text-field
                        label="Cost Price"
                    ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6" md="3">
                    <v-select
                    :items="vehicles"
                    label="Vehicle"
                    ></v-select>
                </v-col>
                <v-col cols="12" sm="6" md="3">
                    <v-select
                    :items="categories"
                    label="Category"
                    ></v-select>
                </v-col>
            </v-row>
        </v-container>
    </v-form>
 </v-card>
</template>
<script>
  export default {
    data: () => ({
      units: ['Each', 'Gallon', 'Bottle', 'Other'],
      vehicles: ['Opel', 'Chevrolet', 'Other'],
      categories: ['Mansour Parts', 'Non Mansour Parts', 'Oils', 'Other'],
    }),
  }
</script>