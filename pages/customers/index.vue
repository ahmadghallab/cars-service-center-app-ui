<template>
    <div>
        <v-skeleton-loader
            class="mx-auto"
            type="table"
            v-if="fetchingCustomers"
        ></v-skeleton-loader>
        <v-card v-else>
            <v-card-text>
                <v-data-table
                    :loading="searchingCustomers" loading-text="Loading... Please wait"
                    :headers="headers"
                    :items="customers"
                    :search="search"
                    :server-items-length="pagination.total"
                    hide-default-footer
                    :fixed-header="true"
                    >
                    <template v-slot:top>
                        <v-row>
                            <v-col>
                                <v-dialog v-model="dialog" max-width="500px">
                                    <template v-slot:activator="{ on }">
                                        <v-btn text v-on="on">New Customer</v-btn>
                                    </template>
                                    <v-card>
                                        <v-card-title  class="headline">{{ formTitle }}</v-card-title>
                                        <v-card-text>
                                            <v-row>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-select
                                                        :items="titles"
                                                        v-model="editedCustomer.title"
                                                        label="Title"
                                                    ></v-select>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedCustomer.name" label="Name"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedCustomer.phone" label="Phone"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedCustomer.alt_phone" label="Alt Phone"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedCustomer.email" label="Email"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedCustomer.address" label="Address"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-select
                                                        :items="classes"
                                                        v-model="editedCustomer.class"
                                                        label="Class"
                                                    ></v-select>
                                                </v-col>
                                            </v-row>
                                        </v-card-text>
                                        <v-divider />
                                        <v-card-actions>
                                            <v-spacer></v-spacer>
                                            <v-btn text @click="close">Cancel</v-btn>
                                            <v-btn color="orange" text @click="save" :loading="saving" :disabled="customerFormValidator">Save</v-btn>
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
                                    @input="searchCustomer()"
                                ></v-text-field>
                            </v-col>
                        </v-row>
                        <v-dialog
                            v-model="deleteDialog" 
                            max-width="290"
                            >
                            <v-card>
                                <v-card-title class="headline">Delete Customer</v-card-title>

                                <v-card-text>
                                You are about to delete this customer <strong> {{ deletedCustomer.name }} </strong>. No one will be able to access this customer ever again.
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
                                    @click="deleteCustomer()"
                                    :loading="deletingCustomer"
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
                            @click="editCustomer(item)"
                        >
                            edit
                        </v-btn>
                        <v-btn
                            small
                            text
                            @click="deleteCustomerConfirm(item)"
                            >
                            delete
                        </v-btn>
                    </template>
                    <template v-if="pagination.lastPage > 1" v-slot:footer>
                        <!-- <div class="text-right">
                            <v-pagination
                                v-model="pagination.current"
                                :length="pagination.lastPage"
                                @input="onPageChange"
                                circle
                            ></v-pagination>
                        </div> -->
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
import debounce from 'lodash/debounce';

export default {
    data () {
        return {
            dialog: false,
            deleteDialog: false,
            fetchingCustomers: true,
            searchingCustomers: false,
            deletingCustomer: false,
            saving: false,
            pagination: {
                current: 1,
                total: 0,
                lastPage: 0
            },
            search: '',
            headers: [
                { text: 'CustId', value: 'id'},
                {
                    text: 'Title',
                    align: 'left',
                    sortable: false,
                    value: 'title',
                },
                { text: 'Name', value: 'name' },
                { text: 'Phone', value: 'phone', sortable: false },
                { text: 'Address', value: 'address', sortable: false },
                { text: 'Class', value: 'class' },
                { text: 'Actions', value: 'action', sortable: false },
            ],
            customers: [],
            editedIndex: -1,
            deletedIndex: -1,
            deletedCustomer: {
                id: null,
                name: null
            },
            editedCustomer: {
                id: null,
                title: null,
                name: null,
                phone: null,
                alt_phone: null,
                email: null,
                address: null,
                class: null,
            },
            defaultCustomer: {
                id: null,
                title: null,
                name: null,
                phone: null,
                alt_phone: null,
                email: null,
                address: null,
                class: null,
            },
            titles: ['Mr', 'Mrs', 'Ms', 'Dr', 'Eng', 'Company', 'Individual'],
            classes: ['A', 'B', 'C']
        }
    },
    computed: {
        formTitle () {
            return this.editedIndex === -1 ? 'New Customer' : 'Edit Customer'
        },
        customerFormValidator () {
            return (this.editedCustomer.name && this.editedCustomer.phone) ? false : true
        }
    },

    watch: {
        dialog (val) {
            val || this.close()
        },
    },
    methods: {
        async getCustomers() {
            try {
                let queryParam = `?page=${this.pagination.current}`
                if (this.search) {
                    queryParam += `&q=${this.search}`
                }
                let response = await this.$axios.$get(`/customers${queryParam}`);
                this.customers = response.data   
                this.pagination.total = response.total           
                this.pagination.current = response.current_page           
                this.pagination.lastPage = response.last_page  
                this.fetchingCustomers = false
                if (this.searchingCustomers) this.searchingCustomers = false
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
            this.fetchingCustomers = true
            this.getCustomers();
        },
        searchCustomer: debounce(function () {
            this.pagination.current = 1
            this.searchingCustomers = true
            this.getCustomers()
        }, 1300),
        editCustomer (item) {
            this.editedIndex = this.customers.indexOf(item)
            this.editedCustomer = Object.assign({}, item)
            this.dialog = true
        },

        deleteCustomerConfirm (item) {
            this.deletedIndex = this.customers.indexOf(item)
            this.deletedCustomer = Object.assign({}, item)
            this.deleteDialog = true            
        },

        async deleteCustomer () {
            this.deletingCustomer = true
            try {
                await this.$axios.$delete(`/customers/${this.deletedCustomer.id}`)
                this.customers.splice(this.deletedIndex, 1)
                this.deleteDialog = false
                this.deletingCustomer = false
            } catch (e) {
                console.log(e);
            }
        },

        close () {
            this.dialog = false
            setTimeout(() => {
                this.editedCustomer = Object.assign({}, this.defaultCustomer)
                this.editedIndex = -1
            }, 300)
        },

        async save () {
            this.saving = true
            if (this.editedIndex > -1) {
                try {
                    await this.$axios.$patch(`/customers/${this.editedCustomer.id}`, this.editedCustomer) 
                    Object.assign(this.customers[this.editedIndex], this.editedCustomer)
                } catch (e) {

                }
            } else {
                try {
                    const response = await this.$axios.$post('/customers', this.editedCustomer) 
                    this.customers.push(response.data)
                } catch (e) {

                } 
            }
            this.saving = false
            this.close()
        },
    },
    mounted () {
        this.getCustomers()
    }
}
</script>