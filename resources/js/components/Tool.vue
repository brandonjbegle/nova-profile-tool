<template>
    <loading-view :loading="loading">
        <heading class="mb-3">{{ __("Update Profile") }}</heading>

        <card class="overflow-hidden">
            <form @submit.prevent="saveProfile">
                <!-- Validation Errors -->
                <validation-errors :errors="validationErrors"/>

                <!-- Fields -->
                <div v-for="field in fields">
                    <component
                        :is="'form-' + field.component"
                        :errors="validationErrors"
                        :resource-name="resourceName"
                        :field="field"
                        :field-name="field.attribute"
                        :via-resource="viaResource"
                        :via-resource-id="viaResourceId"
                        :via-relationship="viaRelationship"
                    />
                </div>

                <!-- Create Button -->
                <div class="bg-30 flex px-8 py-4">
                    <button dusk="create-and-add-another-button" class="ml-auto btn btn-default btn-primary mr-3">
                        {{ __('Save Profile') }}
                    </button>
                </div>
            </form>
        </card>
    </loading-view>
</template>

<script>
import {Errors, Minimum} from 'laravel-nova'

export default {
    metaInfo() {
        return {
            title: 'Profile',
        }
    },
    data: () => ({
        loading: true,
        fields: [],
        validationErrors: new Errors(),
    }),

    created() {
        this.getFields()
    },

    methods: {
        /**
         * Get the available fields for the resource.
         */
        async getFields() {
            this.fields = []

            const {data: fields} = await Nova.request().get(
                '/nova-vendor/nova-profile-tool/'
            )

            this.fields = fields
            this.loading = false
        },

        /**
         * Saves the user's profile
         */
        async saveProfile() {
            this.loading = true;

            this.createRequest()
                .then(() => {
                    this.loading = false;
                    this.$toasted.show(
                        this.__('Your profile has been saved!'),
                        {type: 'success'}
                    )
                    this.getFields()
                    this.validationErrors = new Errors()
                })
                .catch(error => {
                    this.loading = false;
                    if (error.response.status === 422) {
                        this.validationErrors = new Errors(error.response.data.errors)
                    }
                    this.$toasted.show(
                        this.__('There was an error submitting the form.'),
                        {type: 'error'}
                    )
                })
        },

        /**
         * Send a create request to update the user's profile data
         */
        createRequest() {
            return Nova.request().post(
                '/nova-vendor/nova-profile-tool',
                this.createResourceFormData()
            )
        },

        /**
         * Create the form data for creating the resource.
         */
        createResourceFormData() {
            return _.tap(new FormData(), formData => {
                _.each(this.fields, field => {
                    field.fill(formData)
                })
            })
        },
    },
}
</script>
