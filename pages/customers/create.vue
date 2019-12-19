<template>
 <v-card
    outlined
  >
    <v-form v-on:submit.prevent="postCustomer()">
        <v-card-text>
            <v-container>
                <v-row>
                    <v-col cols="12" sm="6" md="3">
                        <v-select
                            :items="titles"
                            v-model="customer.title"
                            label="Title"
                        ></v-select>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Name"
                            v-model="customer.name"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Phone"
                            v-model="customer.phone"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Alt Phone"
                            v-model="customer.alt_phone"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Address"
                            v-model="customer.address"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Email"
                            v-model="customer.email"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-select
                            :items="classes"
                            v-model="customer.class"
                            label="Class"
                        ></v-select>
                    </v-col>
                </v-row>
            </v-container>
        </v-card-text>
        <v-divider />
        <v-card-text>
            <v-btn type="submit" :loading="customerPosting" :disabled="postCustomerValidator">Save</v-btn>
        </v-card-text>
    </v-form>
 </v-card>
</template>
<script>
export default {
    data () {
        return {
            customerPosting: false,
            customer: {
                'title': null,
                'name': null,
                'phone': null,
                'alt_phone': null,
                'address': null,
                'email': null,
                'class': null
            },
            titles: ['Mr', 'Mrs', 'Ms', 'Dr', 'Eng', 'Company', 'Individual'],
            classes: ['A', 'B', 'C']
        }
    },
    computed: {
        postCustomerValidator () {
        return (this.customer.name && this.customer.phone) ? false : true
        }
    },
    methods: {
       async postCustomer() {
            this.customerPosting = true
            try {
                let response = await this.$axios.$post('/customers', this.customer)
                this.customerPosting = false
            } catch (e) {
            }
        } 
    },
    created () {
        
    }
}
</script>