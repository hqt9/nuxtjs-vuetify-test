<template>
    <div>
        <v-form>
            <v-row>
                <v-col cols="12" md="4">
                    <v-text-field
                        outlined
                        v-model="searchForm.author"
                        :counter="50"
                        :maxLength="50"
                        label="Author"
                    ></v-text-field>
                </v-col>

                <v-col cols="12" md="4">
                    <v-text-field
                        outlined
                        v-model="searchForm.summary"
                        :counter="100"
                        :maxLength="100"
                        label="Summary"
                    ></v-text-field>
                </v-col>

                <!-- <v-col cols="12" md="4">
                    <v-date-picker
                        v-model="searchForm.published_at"
                        range
                        label="Published At"
                    ></v-date-picker>
                    <v-daterange
                        :options="dateRangeOptions"
                        @input="onDateRangeChange"
                    ></v-daterange>
                </v-col> -->
            </v-row>
        </v-form>
        <v-btn elevation="3" color="primary" @click="search">Search</v-btn>
        <v-btn elevation="3" color="primary" @click="reset">Reset</v-btn>
    </div>
</template>

<script>
export default {
    props: {
        searchForm: {}
    },
    methods: {
        search() {
            this.$emit('search')
        },
        reset() {
            this.$emit('reset')
        }
    }
}
</script>