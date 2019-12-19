<template>
    <div>
        <v-skeleton-loader
            class="mx-auto"
            type="card"
            v-if="loadingAllMakes"
        ></v-skeleton-loader>
        <v-card v-else outlined>
            <v-list>
                <v-subheader v-if="!makes.length" class="text-center">No Vehicles Yet</v-subheader>
                <v-list-item v-else
                    v-for="(item, index) in makes"
                    :key="item"
                    @click=""
                    :to="`/makes/${item.id}`"
                > 
                    <v-list-item-content>
                        <v-list-item-title v-text="item.name"></v-list-item-title>
                    </v-list-item-content>
                </v-list-item>
            </v-list>
            <v-divider />
            <v-card-text>
                <v-form v-on:submit.prevent="postMake()">
                    <v-text-field
                        label="Make Name"
                        v-model="name"
                    ></v-text-field>
                    <v-btn type="submit" :loading="saving">Save</v-btn>
                </v-form>
            </v-card-text>
        </v-card>
    </div>
</template>
<script>
export default {
    data: () => ({
        name: null,
        saving: false,
        loadingAllMakes: true,
        makes: []
    }),
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
        async postMake() {
            this.saving = true
            try {
                let response = await this.$axios.$post('/makes', {'name': this.name});
                this.makes.push(response);
                this.saving = false
                this.name = null
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