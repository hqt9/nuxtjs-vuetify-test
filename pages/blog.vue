<template>
    <div>
        <div class="mb-6">
            <search
                :searchForm="searchForm"
                @search="search"
                @reset="reset"
            ></search>
        </div>

        <v-data-table
            :headers="headers"
            :items="desserts"
            :options.sync="options"
            :server-items-length="total"
            :loading="loading"
            class="elevation-1"
        >
            <template v-slot:[`item.image`]="{ item }">
                <v-img
                    class="ma-1"
                    :aspect-ratio="16 / 9"
                    :width="200"
                    :src="'images/' + item.image"
                ></v-img>
            </template>

            <template v-slot:top>
                <v-toolbar flat>
                    <v-toolbar-title>Blog List</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-spacer></v-spacer>

                    <v-dialog v-model="dialog" max-width="800px">
                        <template v-slot:activator="{ on, attrs }">
                            <v-btn
                                color="primary"
                                dark
                                class="mb-2"
                                v-bind="attrs"
                                v-on="on"
                            >
                                New Blog
                            </v-btn>
                        </template>

                        <v-card>
                            <v-card-title>
                                <span class="text-h5">{{ formTitle }}</span>
                            </v-card-title>

                            <v-card-text>
                                <v-form
                                    ref="form"
                                    v-model="valid"
                                    lazy-validation
                                >
                                    <v-container>
                                        <v-row>
                                            <v-col cols="12" sm="12" md="12">
                                                <v-text-field
                                                    outlined
                                                    v-model="editedItem.author"
                                                    :rules="authorRules"
                                                    label="Author"
                                                    required
                                                ></v-text-field>
                                            </v-col>
                                            <v-col cols="12" sm="12" md="12">
                                                <v-text-field
                                                    outlined
                                                    v-model="editedItem.summary"
                                                    :rules="summaryRules"
                                                    label="Summary"
                                                    required
                                                ></v-text-field>
                                            </v-col>
                                            <v-col cols="12" sm="12" md="12">
                                                <v-textarea
                                                    outlined
                                                    v-model="editedItem.content"
                                                    :rules="contentRules"
                                                    label="Content"
                                                    required
                                                ></v-textarea>
                                            </v-col>
                                            <v-col cols="12" sm="12" md="12">
                                                <v-file-input
                                                    v-model="editedItem.images"
                                                    chips
                                                    accept="image/*"
                                                    outlined
                                                    label="Image"
                                                    prepend-icon=""
                                                    @change="previewImg($event)"
                                                ></v-file-input>
                                                <div style="width: 400px">
                                                    <img
                                                        v-if="photoSrc"
                                                        :src="photoSrc"
                                                        style="width: 100%"
                                                    />
                                                </div>
                                            </v-col>
                                        </v-row>
                                    </v-container>
                                </v-form>
                            </v-card-text>

                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn
                                    color="blue darken-1"
                                    text
                                    @click="close"
                                >
                                    Cancel
                                </v-btn>
                                <v-btn color="blue darken-1" text @click="save">
                                    Save
                                </v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>

                    <v-dialog v-model="dialogDelete" max-width="500px">
                        <v-card>
                            <v-card-title class="text-h5"
                                >Are you sure you want to delete this
                                blog?</v-card-title
                            >
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn
                                    color="blue darken-1"
                                    text
                                    @click="closeDelete"
                                    >Cancel</v-btn
                                >
                                <v-btn
                                    color="blue darken-1"
                                    text
                                    @click="deleteItemConfirm"
                                    >OK</v-btn
                                >
                                <v-spacer></v-spacer>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>

            <!-- eslint-disable-next-line -->
            <template v-slot:item.actions="{ item }">
                <!-- <v-icon small class="mr-2" @click="showItem(item)">
                    mdi-eye
                </v-icon> -->
                <v-icon small class="mr-2" @click="editItem(item)">
                    mdi-pencil
                </v-icon>
                <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
            </template>

            <!-- <template v-slot:no-data>
                <v-btn color="primary" @click="reset"> Reset </v-btn>
            </template> -->
        </v-data-table>
    </div>
</template>
  
  
<script>
import Search from '../components/Search'

const searchForm = {
    author: '',
    summary: '',
    published_start: '',
    published_end: '',
    published_at: []
}

const newBlog = {
    author: '',
    summary: '',
    image: '',
    images: [],
    content: ''
}

export default {
    components: { Search },

    data: () => ({
        photoSrc: '',
        valid: false,
        authorRules: [
            (v) => !!v || 'Author is required',
            (v) =>
                (v && v.length <= 50) ||
                'Author must be less than 50 characters'
        ],
        summaryRules: [
            (v) => !!v || 'Summary is required',
            (v) =>
                (v && v.length <= 100) ||
                'Summary must be less than 100 characters'
        ],
        contentRules: [(v) => !!v || 'Content is required'],
        dialog: false,
        dialogDelete: false,
        searchForm: Object.assign({}, searchForm),
        headers: [
            {
                text: 'Author',
                align: 'start',
                value: 'author',
                sortable: false
            },
            {
                text: 'Summary',
                align: 'start',
                value: 'summary',
                sortable: false
            },
            {
                text: 'Image',
                align: 'start',
                value: 'image',
                sortable: false
            },
            {
                text: 'Content',
                align: 'start',
                value: 'content',
                sortable: false
            },
            {
                text: 'Published At',
                align: 'start',
                value: 'updated_at',
                sortable: false
            },
            {
                text: 'Actions',
                value: 'actions',
                sortable: false
            }
        ],
        desserts: [],
        editedIndex: -1,
        editedItem: Object.assign({}, newBlog),
        total: 0,
        options: {
            page: 1,
            itemsPerPage: 5
        },
        loading: true
    }),

    computed: {
        formTitle() {
            return this.editedIndex === -1 ? 'New Blog' : 'Edit Blog'
        }
    },

    watch: {
        dialog(val) {
            val || this.close()
        },
        dialogDelete(val) {
            val || this.closeDelete()
        },
        options: {
            handler() {
                this.initialize()
            },
            deep: false
        }
    },

    // mounted () {
    //   this.initialize()
    // },

    methods: {
        search() {
            this.initialize()
        },
        reset() {
            this.searchForm = Object.assign({}, searchForm)

            if (this.options.page == 1) {
                this.initialize()
            } else {
                this.options.page = 1
            }
        },
        async initialize() {
            const { page, itemsPerPage } = this.options

            let data = {
                params: {
                    author: this.searchForm.author,
                    summary: this.searchForm.summary,
                    page: page,
                    pageSize: itemsPerPage
                }
            }
            const res = await this.$axios.$get('/blogs', data)
            this.loading = false
            if (res.code == 200) {
                this.desserts = res.data.list
                this.total = res.data.total
            } else {
                this.desserts = []
                this.total = 0
            }
        },
        showItem(item) {
            this.editedIndex = item.id
            this.editedItem = Object.assign({}, item)
            this.dialog = true
        },
        editItem(item) {
            this.editedIndex = item.id
            this.editedItem = Object.assign({}, item)
            this.photoSrc = 'images/' + item.image
            this.dialog = true
        },
        close() {
            this.dialog = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, newBlog)
                this.editedIndex = -1
                this.$refs.form.reset()
            })
        },
        save() {
            if (this.$refs.form.validate()) {
                // console.log(this.editedItem)

                const formData = new FormData()

                formData.append('author', this.editedItem.author)
                formData.append('summary', this.editedItem.summary)
                formData.append('content', this.editedItem.content)
                formData.append('image', this.editedItem.image)
                formData.append('images', this.editedItem.images)

                if (this.editedItem.id) {
                    formData.append('id', this.editedItem.id)
                }

                this.$axios({
                    method: 'POST',
                    url: '/blogs',
                    data: formData,
                    headers: {
                        'Content-Type': 'multipart/form-data'
                    }
                }).then((res) => {
                    // console.log(res)
                    this.close()
                    this.search()
                })
            }
        },
        deleteItem(item) {
            this.editedIndex = this.desserts.indexOf(item)
            this.editedItem = Object.assign({}, item)
            this.dialogDelete = true
        },
        deleteItemConfirm() {
            let data = {
                params: this.editedItem
            }
            this.$axios.$delete('/blogs', data).then((res) => {
                this.closeDelete()
                this.search()
            })
        },
        closeDelete() {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, newBlog)
                this.editedIndex = -1
                this.$refs.form.reset()
            })
        },
        previewImg(e) {
            const files = e
            if (files && (files.length > 0 || files.name != undefined)) {
                this.photoSrc = URL.createObjectURL(files)
                // URL.revokeObjectURL(files)
            } else {
                this.photoSrc = ''
            }
        }
    }
}
</script>