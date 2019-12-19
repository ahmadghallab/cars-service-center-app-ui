<template>
    <div>
        <v-skeleton-loader
            class="mx-auto"
            type="table"
            v-if="fetchingCustomers"
        ></v-skeleton-loader>
        <v-card v-else>
            <v-card-title>
                <v-btn text>New Customer</v-btn>
                <v-spacer></v-spacer>
                <v-text-field
                    v-model="search"
                    label="Search"
                    single-line
                    hide-details
                ></v-text-field>
            </v-card-title>
            <v-card-text>
                <v-data-table
                :loading="fetchingMoreCustomers" loading-text="Loading... Please wait"
                :headers="headers"
                :items="customers"
                :search="search"
                ></v-data-table>
            </v-card-text>  
            <!-- <v-card-text>
                <v-simple-table fixed-header>
                    <template v-slot:default>
                    <thead>
                        <tr>
                        <th class="text-left">Title</th>
                        <th class="text-left">Name</th>
                        <th class="text-left">Phone</th>
                        <th class="text-left">Address</th>
                        <th class="text-left">Class</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(customer, index) in customers" :key="index" >
                            <td>{{ customer.title }}</td>
                            <td>{{ customer.name }}</td>
                            <td>{{ customer.phone }}</td>
                            <td>{{ customer.address }}</td>
                            <td>{{ customer.class }}</td>
                        </tr>
                    </tbody>
                    </template>
                </v-simple-table>
            </v-card-text> -->
        </v-card>
    </div>
</template>
<script>
export default {
    data () {
        return {
            fetchingCustomers: true,
            fetchingMoreCustomers: false,
            search: '',
            headers: [
                {
                    text: 'Title',
                    align: 'left',
                    sortable: false,
                    value: 'title',
                },
                { text: 'Name', value: 'name' },
                { text: 'Phone', value: 'phone' },
                { text: 'Alt Phone', value: 'alt_phone' },
                { text: 'Address', value: 'address' },
                { text: 'Class', value: 'class' },
            ],
            customers: []
        }
    },
    methods: {
        async getCustomers() {
            try {
                let response = await this.$axios.$get('/customers');
                this.customers = response
                console.log(response)
                
                this.fetchingCustomers = false
            } catch (e) {
                console.log(e);
            }
        }
    },
    created () {
        this.getCustomers()
    }
}
</script>