<template>
    <div>
        <v-skeleton-loader
            class="mx-auto mb-3"
            type="card"
            v-if="loadingGetMake"
        ></v-skeleton-loader>
        <v-card v-else class="mb-3" outlined>
            <v-card-text>
                <v-form v-on:submit.prevent="updateMake()">
                    <v-text-field
                        label="Make Name"
                        v-model="makeName"
                    ></v-text-field>
                    <v-btn type="submit" :loading="updateSaving">Save</v-btn>
                </v-form>
            </v-card-text>
        </v-card>

        <v-skeleton-loader
            class="mx-auto"
            type="card"
            v-if="loadingGetAllModels"
        ></v-skeleton-loader>
        <v-card class="mb-3" v-else outlined>
            <v-list>
                <v-subheader v-if="!models.length" class="text-center">No models for this make yet</v-subheader>
                <v-list-item v-else
                    v-for="(model, index) in models"
                    :key="index"
                    @click="triggerEditModelDialog(model.id)"
                    > 
                    <v-list-item-content>
                        <v-list-item-title v-text="model.name"></v-list-item-title>
                    </v-list-item-content>
                    <div v-if="editModelId == model.id">
                        <v-dialog
                            v-model="editModelDialog" 
                            max-width="290"
                            >
                            <v-card>
                                <v-card-title class="headline">Update Model</v-card-title>
                                <v-card-text>
                                    <v-text-field
                                        label="Model Name"
                                        v-model="model.name"
                                    ></v-text-field>
                                </v-card-text>
                                <v-divider />
                                <v-card-actions>
                                    <v-spacer></v-spacer>

                                    <v-btn
                                        text
                                        @click="editModelDialog = false"
                                    >
                                        Dismiss
                                    </v-btn>
                                    <v-btn
                                        color="pink"
                                        text
                                        @click="updateModel(index, model.id)"
                                        class="font-weight-bold"
                                        :loading="updatingModel"
                                        >
                                        Save
                                    </v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                    </div>
                </v-list-item>
            </v-list>
            <v-divider />
            <v-card-text class="bg-black">
                <v-form v-on:submit.prevent="postModel()">
                    <v-text-field
                        label="Model Name"
                        v-model="modelName"
                    ></v-text-field>
                    <v-btn type="submit" :loading="postSaving">Save</v-btn>
                </v-form>
            </v-card-text>
        </v-card>
        <div v-if="!loadingGetAllModels && !models.length">
            <v-btn
                @click.stop="deleteDialog = true"
                >
                Delete this make
            </v-btn>

            <v-dialog
                v-model="deleteDialog" 
                max-width="290"
                >
                <v-card>
                    <v-card-title class="headline">{{ makeName }}</v-card-title>

                    <v-card-text>
                    You are about to delete this make. No one will be able to access this make ever again.
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
                        @click="deleteMake()"
                        class="font-weight-bold"
                        :loading="deletingMake"
                    >
                        Yes Delete
                    </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </div>
    </div>
</template>
<script>
export default {
    data () {
        return {
            id: this.$route.params.id,
            deleteDialog: false,
            editModelDialog: false,
            editModelId: null,
            makeName: null,
            modelName: null,
            updateSaving: false,
            postSaving: false,
            deletingMake: false,
            updatingModel: false,
            loadingGetMake: true,
            loadingGetAllModels: true,
            models: []
        }
    },
    methods: {
        triggerEditModelDialog (modelId) {
            this.editModelDialog = true
            this.editModelId = modelId
        },
        async getMake() {
            try {
                let response = await this.$axios.$get(`/makes/${this.id}`);
                this.makeName = response.name
                this.loadingGetMake = false
            } catch (e) {
                console.log(e);
            }
        },
        async getModelsByMake() {
            try {
                let response = await this.$axios.$get(`/makes/${this.id}/models`);
                this.models = response
                this.loadingGetAllModels = false
            } catch (e) {
                console.log(e);
            }
        },
        async updateMake() {
            this.updateSaving = true
            try {
                let response = await this.$axios.$patch(`/makes/${this.id}`, {'name': this.makeName});
                this.updateSaving = false
            } catch (e) {
                console.log(e);
            }
        },
        async postModel() {
            this.postSaving = true
            try {
                let response = await this.$axios.$post('/models', 
                    {
                        'name': this.modelName,
                        'make': this.id
                    }
                )
                this.models.push(response)
                this.postSaving = false
                this.modelName = null
            } catch (e) {
                console.log(e);
            }
        },
        async deleteMake() {
            this.deletingMake = true
            try {
                let response = await this.$axios.$delete(`/makes/${this.id}`)
                this.$router.push('vehicle')
            } catch (e) {
                console.log(e);
            }
        },
        async updateModel(idx, modelId) {            
            this.updatingModel = true
            try {
                let response = await this.$axios.$patch(`/models/${modelId}`, 
                    {
                        'name': this.models[idx].name
                    }
                );
                this.updatingModel = false
                this.editModelDialog = false
            } catch (e) {
                console.log(e);
            }
        },
    },
    created () {
        this.getMake()
        this.getModelsByMake()
    }
}
</script>