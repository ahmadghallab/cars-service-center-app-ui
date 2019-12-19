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
                    :loading="fetchingMoreCustomers" loading-text="Loading... Please wait"
                    :headers="headers"
                    :items="customers"
                    :search="search"
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
                                                        v-model="editedItem.title"
                                                        label="Title"
                                                    ></v-select>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedItem.name" label="Name"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedItem.phone" label="Phone"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedItem.alt_phone" label="Alt Phone"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedItem.email" label="Email"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-text-field v-model="editedItem.address" label="Address"></v-text-field>
                                                </v-col>
                                                <v-col cols="12" sm="6" md="4">
                                                    <v-select
                                                        :items="classes"
                                                        v-model="editedItem.class"
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
                                You are about to delete this customer <strong> {{ deletedItem.name }} </strong>. No one will be able to access this customer ever again.
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
                            @click="editItem(item)"
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
                </v-data-table>
            </v-card-text>
        </v-card>
    </div>
</template>
<script>
export default {
    data () {
        return {
            dialog: false,
            deleteDialog: false,
            fetchingCustomers: true,
            fetchingMoreCustomers: false,
            deletingCustomer: false,
            saving: false,
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
            deletedItem: {
                id: null,
                name: null
            },
            editedItem: {
                id: null,
                title: null,
                name: null,
                phone: null,
                alt_phone: null,
                email: null,
                address: null,
                class: null,
            },
            defaultItem: {
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
            return (this.editedItem.name && this.editedItem.phone) ? false : true
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
                let response = await this.$axios.$get('/customers');
                this.customers = response                
                this.fetchingCustomers = false
            } catch (e) {
                console.log(e);
            }
        },
        editItem (item) {
            this.editedIndex = this.customers.indexOf(item)
            this.editedItem = Object.assign({}, item)
            this.dialog = true
        },

        deleteCustomerConfirm (item) {
            this.deletedIndex = this.customers.indexOf(item)
            this.deletedItem = Object.assign({}, item)
            this.deleteDialog = true            
        },

        async deleteCustomer () {
            this.deletingCustomer = true
            try {
                await this.$axios.$delete(`/customers/${this.deletedItem.id}`)
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
                this.editedItem = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            }, 300)
        },

        async save () {
            this.saving = true
            if (this.editedIndex > -1) {
                try {
                    await this.$axios.$patch(`/customers/${this.editedItem.id}`, this.editedItem) 
                    Object.assign(this.customers[this.editedIndex], this.editedItem)
                } catch (e) {

                }
            } else {
                try {
                    const response = await this.$axios.$post('/customers', this.editedItem) 
                    this.customers.push(response)
                } catch (e) {

                } 
            }
            this.saving = false
            this.close()
        },
    },
    created () {
        this.getCustomers()
    }
}
</script>