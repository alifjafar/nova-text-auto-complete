<template>
    <r64-default-field :hide-field="hideField"
                       :field="field"
                       :hide-label="hideLabelInForms"
                       :field-classes="fieldClasses"
                       :wrapper-classes="wrapperClasses"
                       :label-classes="labelClasses">
        <template slot="field">
            <text-auto-complete
                    @input="performSearch"
                    @selected="selectItem"
                    :error="hasError"
                    :data='availableItems'
                    v-model="value"
                    :extra-attributes="extraAttributes"
            ></text-auto-complete>
            <p v-if="hasError" class="my-2 text-danger">
                {{ firstError }}
            </p>
        </template>
    </r64-default-field>
</template>

<script>
    import storage from "../storage/TextItemStorage"
    import {FormField, HandlesValidationErrors} from 'laravel-nova'
    import TextAutoComplete from "./TextAutoComplete";
    import R64DefaultField from "./DefaultField";
    import R64Field from '../mixins/R64Field'

    export default {
        props: ['resourceId'],

        components: {TextAutoComplete, R64DefaultField},

        mixins: [FormField, HandlesValidationErrors, R64Field],

        data: () => ({
            availableItems: [],
        }),

        methods: {
            /**
             * Perform search
             */
            performSearch(search) {
                const trimmedSearch = search.trim()

                if (trimmedSearch == '') {
                    return;
                }

                if (this.field.items) {
                    this.availableItems = this.field.items.filter(item => item.toLowerCase().indexOf(trimmedSearch.toLowerCase()) > -1);
                } else {
                    storage.fetchAvailableItems(this.resourceName, this.resourceId, this.field.attribute, {params: {search: trimmedSearch}})
                        .then(({data: items}) => {
                            this.availableItems = items;
                        });
                }
            },

            /**
             * Select item
             */
            selectItem(item) {
                this.value = item;
            }
        },

        computed: {
            defaultAttributes() {
                return {
                    type: this.field.type || 'text',
                    min: this.field.min,
                    max: this.field.max,
                    step: this.field.step,
                    pattern: this.field.pattern,
                    placeholder: this.field.placeholder || this.field.name,
                    class: this.errorClasses,
                }
            },

            extraAttributes() {
                const attrs = this.field.extraAttributes

                return {
                    // Leave the default attributes even though we can now specify
                    // whatever attributes we like because the old number field still
                    // uses the old field attributes
                    ...this.defaultAttributes,
                    ...attrs,
                }
            },
        },
    }
</script>
