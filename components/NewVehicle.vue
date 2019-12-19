<template>
 <v-card
    outlined
  >
    <v-form>
        <v-card-text>
            <v-container>
                <v-row>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Vin Number"
                            v-model="vehicle.vin"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Engine Number"
                            v-model="vehicle.engine"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Licence Plate"
                            v-model="vehicle.licencePlate"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-select
                        :items="makes"
                        v-model="vehicle.make"
                        item-text="name"
                        item-value="id"
                        label="Make"
                        :loading="loadingAllMakes"
                        @change="getModelsByMake()"
                        ></v-select>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-select
                        :items="make_models"
                        v-model="vehicle.make_model"
                        item-text="name"
                        item-value="id"
                        label="Model"
                        :loading="loadingGetAllModels"
                        ></v-select>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Counter"
                            v-model="vehicle.counter"
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="3">
                        <v-text-field
                            label="Customer"
                            v-model="vehicle.customer"
                        ></v-text-field>
                    </v-col>
                </v-row>
            </v-container>
        </v-card-text>
        <v-divider />
        <v-card-text>
            <v-btn type="submit" :loading="vehiclePosting">Save</v-btn>
        </v-card-text>
    </v-form>
 </v-card>
</template>
<script>
export default {
    data () {
        return {
            loadingAllMakes: true,
            loadingGetAllModels: false,
            vehiclePosting: false,
            vehicle: {
                'vin': null,
                'engine': null,
                'licencePlate': null,
                'counter': null,
                'customer': null,
                'make': null,
                'make_model': null
            },
            makes: [],
            make_models: []
        }
    },
    methods: {
        async getAllMakes() {
            try {
                let response = await this.$axios.$get('/makes');
                this.makes = response
                console.log(response)
                
                this.loadingAllMakes = false
            } catch (e) {
                console.log(e);
            }
        },
        async getModelsByMake() {
            let makeId = this.vehicle.make
            this.loadingGetAllModels = true
            try {
                let response = await this.$axios.$get(`/makes/${makeId}/models`);
                this.make_models = response
                this.loadingGetAllModels = false
            } catch (e) {
                console.log(e);
            }
        }
    },
    created () {
        this.getAllMakes()
    }
}
</script>