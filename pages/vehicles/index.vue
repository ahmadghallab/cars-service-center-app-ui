<template>
    <div>
        <v-skeleton-loader
            class="mx-auto"
            type="table"
            v-if="fetchingVehicles"
        ></v-skeleton-loader>
        <v-card v-else>
            <v-card-text>
                <v-data-table
                    :loading="searchingVehicles" loading-text="Loading... Please wait"
                    :headers="headers"
                    :items="vehicles"
                    :server-items-length="pagination.total"
                    hide-default-footer
                    :fixed-header="true"
                    >
                    <template v-slot:top>
                        <v-row>
                            <v-col>
                                <v-dialog v-model="dialog" max-width="500px">
                                    <template v-slot:activator="{ on }">
                                        <v-btn text v-on="on">New Vehicle</v-btn>
                                    </template>
                                    <v-card>
                                        <v-card-title  class="headline">{{ formTitle }}</v-card-title>
                                        <v-card-text>
                                            <v-row>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedVehicle.vin" label="Vin"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedVehicle.engine" label="Engine"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedVehicle.licence_plate" label="Licence Plate"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                  <v-select
                                                  :items="makes"
                                                  v-model="editedVehicle.make.id"
                                                  item-text="name"
                                                  item-value="id"
                                                  label="Make"
                                                  :loading="loadingAllMakes"
                                                  @change="getModelsByMake()"
                                                  ></v-select>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                  <v-select
                                                  :items="make_models"
                                                  v-model="editedVehicle.make_model.id"
                                                  item-text="name"
                                                  item-value="id"
                                                  label="Model"
                                                  :loading="loadingGetAllModels"
                                                  ></v-select>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                  <v-text-field
                                                      label="Counter"
                                                      v-model="editedVehicle.counter"
                                                  ></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="6">
                                                  <v-autocomplete
                                                      v-model="editedVehicle.customer.id"
                                                      :items="customers"
                                                      :loading="fetchingCustomers"
                                                      :search-input.sync="searchCustomerQuery"
                                                      hide-no-data
                                                      hide-selected
                                                      item-text="name"
                                                      item-value="id"
                                                      label="Customer"
                                                  ></v-autocomplete>
                                                </v-col>
                                            </v-row>
                                        </v-card-text>
                                        <v-divider />
                                        <v-card-actions>
                                            <v-spacer></v-spacer>
                                            <v-btn text @click="close">Cancel</v-btn>
                                            <v-btn color="pink accent-2" text @click="save" :loading="saving" :disabled="vehicleFormValidator">Save</v-btn>
                                        </v-card-actions>
                                    </v-card>
                                </v-dialog>
                            </v-col>
                            <v-col>
                                <v-text-field
                                    v-model="search"
                                    label="Search"
                                    single-line
                                    hide-details
                                    @input="searchVehicle()"
                                ></v-text-field>
                            </v-col>
                        </v-row>
                        <v-dialog
                            v-model="deleteDialog" 
                            max-width="290"
                            >
                            <v-card>
                                <v-card-title class="headline">Delete Vehicle</v-card-title>

                                <v-card-text>
                                You are about to delete this vehicle <strong> {{ deletedVehicle.vin }} </strong>. No one will be able to access this vehicle ever again.
                                </v-card-text>

                                <v-card-text>
                                Are you absolutely positive? There's no undo
                                </v-card-text>
                                <v-divider />
                                <v-card-actions>
                                <v-spacer></v-spacer>

                                <v-btn
                                    text
                                    @click="deleteDialog = false"
                                >
                                    Dismiss
                                </v-btn>

                                <v-btn
                                    color="error"
                                    text
                                    @click="deleteVehicle()"
                                    :loading="deletingVehicle"
                                >
                                    Yes Delete
                                </v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                    </template>
                    <template v-slot:item.action="{ item }">
                        <v-btn
                            small
                            text
                            @click="editVehicle(item)"
                        >
                            edit
                        </v-btn>
                        <v-btn
                            small
                            text
                            @click="deleteVehicleConfirm(item)"
                            >
                            delete
                        </v-btn>
                    </template>
                    <template v-if="pagination.lastPage > 1" v-slot:footer>
                        <div class="text-right">
                            <v-btn text icon 
                                :disabled="pagination.current == 1" 
                                @click="onPageChange('prev')">
                                <v-icon>mdi-chevron-left</v-icon>
                            </v-btn>
                            <v-btn text icon 
                                :disabled="pagination.current == pagination.lastPage" 
                                @click="onPageChange('next')">
                                <v-icon>mdi-chevron-right</v-icon>
                            </v-btn>
                        </div>
                    </template>
                </v-data-table>
            </v-card-text>
        </v-card>
    </div>
</template>
<script>

import debounce from 'lodash/debounce'

export default {
    data () {
        return {
            dialog: false,
            deleteDialog: false,
            fetchingVehicles: true,
            searchingVehicles: false,
            deletingVehicle: false,
            saving: false,
            loadingAllMakes: false,
            loadingGetAllModels: false,
            customerNameLimit: 30,
            customers: [],
            fetchingCustomers: false,
            searchCustomerQuery: null,
            pagination: {
                current: 1,
                total: 0,
                lastPage: 0
            },
            search: null,
            headers: [
                { text: 'Vin', value: 'vin', sortable: false },
                { text: 'Engine', value: 'engine', sortable: false },
                { text: 'Licence Plate', value: 'licence_plate', sortable: false },
                { text: 'Counter', value: 'counter', sortable: false },
                { text: 'Customer', value: 'customer.name', sortable: false },
                { text: 'Make', value: 'make.name', sortable: false },
                { text: 'Model', value: 'make_model.name', sortable: false },
                { text: 'Actions', value: 'action', sortable: false },
            ],
            vehicles: [],
            editedIndex: -1,
            deletedIndex: -1,
            deletedVehicle: {
                id: null,
                vin: null
            },
            editedVehicle: {
                id: null,
                vin: null,
                engine: null,
                licence_plate: null,
                counter: null,
                customer: {
                  id: null,
                  name: null
                },
                make: {
                  id: null,
                  name: null
                },
                make_model: {
                  id: null,
                  name: null
                }
            },
            defaultVehicle: {
                id: null,
                vin: null,
                engine: null,
                licence_plate: null,
                counter: null,
                customer: {
                  id: null,
                  name: null
                },
                make: {
                  id: null,
                  name: null
                },
                make_model: {
                  id: null,
                  name: null
                }
            },
            makes: [],
            make_models: []
        }
    },
    computed: {
        formTitle () {
            return this.editedIndex === -1 ? 'New Vehicle' : 'Edit Vehicle'
        },
        vehicleFormValidator () {
          return (this.editedVehicle.vin && this.editedVehicle.engine) ? false : true
        }
    },

    watch: {
        dialog (val) {
          if (val) {
            this.getAllMakes()
          } else {
            this.close()
          }
        },
        searchCustomerQuery (val) {

          if (this.fetchingCustomers) return
         
          this.fetchingCustomers = true

          this.searchCustomer()
        },
    },
    methods: {
        async getVehicles() {
            try {
                let queryParam = `?page=${this.pagination.current}`
                if (this.search) {
                    queryParam += `&q=${this.search}`
                }
                let response = await this.$axios.$get(`/vehicles${queryParam}`);
                this.vehicles = response.data   
                this.pagination.total = response.total           
                this.pagination.current = response.current_page           
                this.pagination.lastPage = response.last_page  
                this.fetchingVehicles = false
                if (this.searchingVehicles) this.searchingVehicles = false
            } catch (e) {
                console.log(e);
            }
        },

        async getAllMakes() {
          this.loadingAllMakes = true
            try {
                let response = await this.$axios.$get('/makes');
                this.makes = response                
                this.loadingAllMakes = false
            } catch (e) {
                console.log(e);
            }
        },

        async getModelsByMake() {
            this.loadingGetAllModels = true
            let makeId = this.editedVehicle.make.id    
            try {
                let response = await this.$axios.$get(`/makes/${makeId}/models`);
                this.make_models = response
                this.loadingGetAllModels = false
            } catch (e) {
                console.log(e);
            }
        },

        onPageChange(dir) {
            if (dir == 'prev') {
                this.pagination.current = this.pagination.current - 1
            } else {
                this.pagination.current = this.pagination.current + 1
            }
            this.fetchingVehicles = true
            this.getVehicles();
        },

        searchVehicle: debounce(function () {
            this.pagination.current = 1
            this.searchingVehicles = true
            this.getVehicles()
        }, 1300),

        searchCustomer: debounce(async function () {
            try {
                if (!this.searchCustomerQuery) this.searchCustomerQuery = this.editedVehicle.customer.id
                let response = await this.$axios.$get(`/customers?q=${this.searchCustomerQuery}`);
                this.customers = response.data 
                this.fetchingCustomers = false
            } catch (e) {
                console.log(e);
            }
        }, 1300),

        editVehicle (item) {
            this.editedIndex = this.vehicles.indexOf(item)
            this.editedVehicle = Object.assign({}, item)
            this.dialog = true
            this.getModelsByMake()
            this.searchCustomer()
        },

        deleteVehicleConfirm (item) {
            this.deletedIndex = this.vehicles.indexOf(item)
            this.deletedVehicle = Object.assign({}, item)
            this.deleteDialog = true            
        },

        async deleteVehicle () {
            this.deletingVehicle = true
            try {
                await this.$axios.$delete(`/vehicles/${this.deletedVehicle.id}`)
                this.vehicles.splice(this.deletedIndex, 1)
                this.deleteDialog = false
                this.deletingVehicle = false
            } catch (e) {
                console.log(e);
            }
        },

        close () {
            this.dialog = false
            setTimeout(() => {
                this.editedVehicle = Object.assign({}, this.defaultVehicle)
                this.editedIndex = -1
            }, 300)
        },

        async save () {
            this.saving = true
            if (this.editedIndex > -1) {
                try {
                    const data = {
                      'vin': this.editedVehicle.vin,
                      'engine': this.editedVehicle.engine,
                      'licence_plate': this.editedVehicle.licence_plate,
                      'counter': this.editedVehicle.counter,
                      'make': this.editedVehicle.make.id,
                      'make_model': this.editedVehicle.make_model.id,
                      'customer': this.editedVehicle.customer.id
                    }
                    
                    await this.$axios.$patch(`/vehicles/${this.editedVehicle.id}`, data) 
                    Object.assign(this.vehicles[this.editedIndex], this.editedVehicle)
                } catch (e) {

                }
            } else {
                try {
                    const data = {
                      'vin': this.editedVehicle.vin,
                      'engine': this.editedVehicle.engine,
                      'licence_plate': this.editedVehicle.licence_plate,
                      'counter': this.editedVehicle.counter,
                      'make': this.editedVehicle.make.id,
                      'make_model': this.editedVehicle.make_model.id,
                      'customer': this.editedVehicle.customer.id
                    }
                    
                    const response = await this.$axios.$post('/vehicles', data) 
                    this.vehicles.push(this.editedVehicle)
                } catch (e) {

                } 
            }
            this.saving = false
            this.close()
        },
    },
    mounted () {
        this.getVehicles()
    }
}
</script>