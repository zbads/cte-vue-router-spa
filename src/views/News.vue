<template>
    <div class="news">
        <div class="modal fade" tabindex="-1" role="dialog" id="create-news-modal">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content" style="padding:20px 30px 30px 30px">
                    <div class="modal-header">
                        <h5 class="text-left modal-title">{{ createNews.title == '' ? 'Create News' : createNews.title }}</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body text-left">
                        <div class="alert alert-success" role="alert" v-show="saveSuccess">News has been successfully saved.</div>

                        <div class="input-group mb-3">
                            <div class="input-group-prepend">
                                <span class="input-group-text">Title</span>
                            </div>
                            <input type="text" class="form-control" v-model="createNews.title" @input="saveSuccess = false" aria-label="Default" aria-describedby="inputGroup-sizing-default">
                        </div>

                        <div style="margin-bottom:10px">
                            <div class="input-group-prepend">
                                <span class="input-group-text" id="basic-addon1">Content</span>
                            </div>
                            <vue-editor 
                                id="createNewsContent"
                                ref="createNewsContentQuillEditor"
                                content-type="html"
                                theme="snow"
                                @textChange="setCreateNewsContent()">
                            </vue-editor>
                        </div>

                        <div class="form-check form-check-inline">
                            <input class="form-check-input" type="checkbox" v-model="createNews.is_published" @change="saveSuccess = false">
                            <label class="form-check-label">
                                Publish?
                            </label>
                        </div>
                    </div>
                    <div class="text-center">
                        <button type="button" class="btn btn-primary" style="margin-right:7px" @click="saveNews()" :disabled="disableSave">Save news</button>
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
                    </div>
                </div>
            </div>
        </div>

        <div class="modal fade" tabindex="-1" role="dialog" id="edit-news-modal">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content" style="padding:20px 30px 30px 30px">
                    <div class="modal-header">
                        <h5 class="modal-title text-left">{{ editNews.title == '' ? 'Edit News' : editNews.title }}</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body text-left">
                        <div class="alert alert-success" role="alert" v-show="updateSuccess">News has been successfully updated.</div>

                        <div class="input-group mb-3">
                            <div class="input-group-prepend">
                                <span class="input-group-text">Title</span>
                            </div>
                            <input type="text" class="form-control" v-model="editNews.title" @input="updateSuccess = false" aria-label="Default" aria-describedby="inputGroup-sizing-default">
                        </div>

                        <div style="margin-bottom:10px">
                            <div class="input-group-prepend">
                                <span class="input-group-text" id="basic-addon1">Content</span>
                            </div>
                            <vue-editor
                                id="editNewsContent"
                                ref="editNewsContentQuillEditor"
                                content-type="html"
                                theme="snow"
                                @textChange="setEditNewsContent()">
                            </vue-editor>
                        </div>

                        <div class="form-check form-check-inline">
                            <input class="form-check-input" type="checkbox" v-model="editNews.is_published" @change="updateSuccess = false">
                            <label class="form-check-label">
                                Publish?
                            </label>
                        </div>
                    </div>
                    <div class="text-center">
                        <button type="button" class="btn btn-primary" style="margin-right:7px" @click="updateNews()" :disabled="disableUpdate">Save changes</button>
                        <button type="button" ref="cancelEditNews" class="btn btn-secondary" data-dismiss="modal" @click="clearEditNewsItem">Cancel</button>
                    </div>
                </div>
            </div>
        </div>

        <div class="row">
            <div class="col-md-10 offset-md-1 col-sm-12">
                <div style="margin-bottom:20px">
                    <h1 class="text-center">
                        <div v-if="loading"  class="spinner-border" role="status" style="margin-right:5px">
                            <span class="sr-only">Loading...</span>
                        </div>
                        News
                    </h1>

                    <div class="text-left">
                        <button class="btn btn-success" data-toggle="modal" data-target="#create-news-modal">+ Create News</button>
                    </div>
                </div>

                <div style="margin-bottom:20px">
                    <h3 class="text-left">Filter</h3>
                    <div class="form-row">
                        <div class="form-group col-md-6 text-left">
                            <label for="titleFilter">Title</label>
                            <input type="text" class="form-control" id="titleFilter" v-model="titleFilter">
                        </div>
                        <div class="form-group col-md-6 text-left">
                            <label for="contentFilter">Content</label>
                            <input type="text" class="form-control" id="contentFilter" v-model="contentFilter">
                        </div>
                    </div>
                    
                    <div class="text-left">
                        <button class="btn btn-primary" @click="loadNews" style="margin-right:5px">Filter results</button>
                        <button class="btn btn-secondary" @click="clearFilters">Clear filter</button>
                    </div>
                </div>

                <div class="news-items">
                    <table class="table table-bordered" width="100%">
                        <thead>
                            <tr style="text-align:left">
                                <th width="25%">Title</th>
                                <th width="50%">Content</th>
                                <th width="15%">Published?</th>
                                <th class="text-center" width="10%">...</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="item in news" :key="item.id" style="text-align:left">
                                <td width="25%">{{ item.title }}</td>
                                <td width="50%" v-html="item.content"></td>
                                <td width="15%">{{ item.is_published ? 'Yes' : 'No' }}</td>
                                <td width="10%" class="text-center">
                                    <button class="btn btn-secondary" @click="loadNewsForEdit(item)" data-toggle="modal" data-target="#edit-news-modal" style="margin-right:5px"><span class="bi bi-pencil"></span></button>
                                    <button class="btn btn-danger" @click="deleteNews(item.id)" data-toggle="modal"><span class="bi bi-trash"></span></button>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import { QuillEditor } from '@vueup/vue-quill'
    import '@vueup/vue-quill/dist/vue-quill.snow.css';

    export default {
        components: {
            'vue-editor': QuillEditor,
        },
        data () {
            return {
                loading: false,
                news: [],
                createNews: {
                    title: '',
                    content: '',
                    is_published: false,
                },
                editNews: {
                    id: 0,
                    title: '',
                    content: '',
                    is_published: false,
                },
                saveSuccess: false,
                updateSuccess: false,
                titleFilter: '',
                contentFilter: '',
            }
        },
        async created () {
            await this.loadNews();

            // Set empty value to 'Create News' content
            this.$refs.createNewsContentQuillEditor.pasteHTML('');  
        },
        computed: {
            disableSave () {
                let title = this.createNews.title || '';
                let content = this.createNews.content || '';

                // Setting empty value to '<p><br></p>' since it's the quill editor default html tags for empty fields
                return title == '' || content == '' || content == '<p><br></p>';
            },
            disableUpdate () {
                let title = this.editNews.title || '';
                let content = this.editNews.content || '';

                // Setting empty value to '<p><br></p>' since it's the quill editor default html tags for empty fields
                return title == '' || content == '' || content == '<p><br></p>';
            }
        },
        methods: {
            async loadNews () {
                this.loading = true

                await this.axios.get('http://localhost:8000/api/news', {
                    headers: { 
                        'Accept': 'application/json', 
                        'Authorization': 'Bearer 2|XDNJTAeAdGXYivRiYNIhw7x2L7edJfV7QqsM68my'
                    },
                    params: {
                        'title': this.titleFilter,
                        'content': this.contentFilter,
                    }
                }).then((response) => {
                    console.log(response)
                    this.news = response.data.data;
                    this.loading = false
                }).catch((error) => {
                    console.log(error);
                });
            },
            loadNewsForEdit (news) {
                this.editNews.id = news.id
                this.editNews.title = news.title
                this.editNews.content = news.content
                this.editNews.is_published = !!news.is_published

                this.$refs.editNewsContentQuillEditor.pasteHTML(news.content);  
            },
            clearCreateNewsItem () {
                this.createNews = {
                    title: '',
                    content: '',
                    is_published: false,
                }

                this.$refs.createNewsContentQuillEditor.pasteHTML('');  
            },
            clearEditNewsItem () {
                this.editNews = {
                    id: 0,
                    title: '',
                    content: '',
                    is_published: false,
                } 

                this.$refs.editNewsContentQuillEditor.pasteHTML('');  
            },
            setCreateNewsContent () {
                this.createNews.content = this.$refs.createNewsContentQuillEditor.getContents();
                this.saveSuccess = false;
            },
            setEditNewsContent () {
                this.editNews.content = this.$refs.editNewsContentQuillEditor.getContents();
                this.updateSuccess = false;
            },
            async saveNews () {
                let ins = this

                await this.axios.post('http://localhost:8000/api/news', this.createNews, {
                    headers: { 
                        'Accept': 'application/json', 
                        'Authorization': 'Bearer 2|XDNJTAeAdGXYivRiYNIhw7x2L7edJfV7QqsM68my'
                    },  
                }).then((response) => {
                    console.log(response)
                    ins.clearCreateNewsItem()
                    ins.saveSuccess = true

                    ins.loadNews()
                }).catch((error) => {
                    console.log(error);
                });
            },
            async updateNews () {
                let ins = this
                let updateUrl = 'http://localhost:8000/api/news/' + this.editNews.id

                await this.axios.patch(updateUrl, this.editNews, {
                    headers: { 
                        'Accept': 'application/json', 
                        'Authorization': 'Bearer 2|XDNJTAeAdGXYivRiYNIhw7x2L7edJfV7QqsM68my'
                    },  
                }).then((response) => {
                    console.log(response)
                    ins.clearEditNewsItem()
                    ins.updateSuccess = true

                    setTimeout(() => this.$refs.cancelEditNews.click(), 1500)

                    ins.loadNews()
                }).catch((error) => {
                    console.log(error);
                });
            },
            async deleteNews (newsId) {
                let ins = this
                let deleteUrl = 'http://localhost:8000/api/news/' + newsId

                await this.axios.delete(deleteUrl, {
                    headers: { 
                        'Accept': 'application/json', 
                        'Authorization': 'Bearer 2|XDNJTAeAdGXYivRiYNIhw7x2L7edJfV7QqsM68my'
                    },  
                }).then((response) => {
                    console.log(response)
                    ins.loadNews()
                }).catch((error) => {
                    console.log(error);
                });
            },
            async clearFilters () {
                this.titleFilter = ''
                this.contentFilter = ''

                this.loadNews()
            },
        }
    }
</script>