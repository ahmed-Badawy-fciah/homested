<template>
    <div>
        
        <h3>Articles</h3>
        <form @submit.prevent="addArticle" class="mb-3">
            <div class="form-group">
                <input type="text" class="form-control" placeholder="Title" v-model="article.title">
            </div>
            <div class="form-group">
                <textarea type="text" class="form-control" placeholder="Body" v-model="article.body"></textarea>
            </div>
            <div class="form-group">
                <button type="submit" class="btn btn-light btn-sm btn-block col-12">Save</button>
                <a @click="clear()" class="btn btn-warning btn-sm btn-block col-12">Clear</a>
            </div>
        </form>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li class="page-item" v-bind:class="[{disabled: !pagination.prev_page_url}]">
                    <a class="page-link" href="#" 
                    @click="fetchArticles(pagination.prev_page_url)">
                        Previous
                    </a>
                </li>   

                
                <li class="page-item disabled">
                    <a class="page-link text-dark" href="#">
                        Page {{pagination.current_page}} of {{pagination.last_page}}
                    </a>
                </li>
                
                <li class="page-item" v-bind:class="[{disabled: !pagination.next_page_url}]">
                    <a class="page-link" href="#" 
                    @click="fetchArticles(pagination.next_page_url)">Next</a>
                </li>
            </ul>
        </nav>

        <div class="card card-body mb-3" v-for="article in articles" v-bind:key="article.id">
            <h3>{{ article.title }}</h3>
            <p>{{ article.body }}</p>
            <button @click="deleteArticle(article.id)" class="btn btn-danger btn-sm mb-1">Delete</button>
            <button @click="editArticle(article)" class="btn btn-primary btn-sm">Edit</button>
        </div>
    </div>
</template>


<script>
    export default {
        data(){
            return{
                articles : [],
                article: {
                    id : '',
                    title : '',
                    body : ''
                },
                article_id : '',
                pagination : {},
                edit : false
            }
        },
        created(){
            this.fetchArticles();
        },
        methods:{
            fetchArticles(page_url){
                let vm = this;
                page_url = page_url || '/api/articles'
                fetch(page_url)
                    .then(res => res.json())
                    .then(res => {
                        this.articles = res.data;
                        vm.makePagination(res.meta , res.links);
                    })
                    .catch(err => console.log(err));
            },
            makePagination(meta , links){
                let pagination = {
                    current_page : meta.current_page,
                    last_page : meta.last_page,
                    next_page_url : links.next,
                    prev_page_url : links.prev
                }
                this.pagination = pagination;
            },
            deleteArticle(id){
                if(confirm('Are you Sure?')){
                    fetch(`api/article/${id}` ,{
                    method: 'delete'})
                    .then(res => res.json())
                    .then(data => {
                        alert(`Article number ${id} is removed!`);
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err));
                }
            },
            addArticle(){
                if(this.edit === false){
                    // Add New Article
                    fetch('api/article' , {
                        method: 'post',
                        body: JSON.stringify(this.article),
                        headers: {
                        'content-type' : 'application/json'
                        }
                    })
                    .then(res => res.json())
                    .then(data => {
                        this.article.title = '',
                        this.article.body = '',
                        alert(`Article Added Successfully!`)
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err));
                }
                else{
                    // Update Exist Article
                    fetch('api/article' , {
                        method: 'put',
                        body: JSON.stringify(this.article),
                        headers: {
                        'content-type' : 'application/json'
                        }
                    })
                    .then(res => res.json())
                    .then(data => {
                        this.article.title = '',
                        this.article.body = '',
                        alert(`Article Updated!`)
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err));
                }
            },
            editArticle(article){
                this.edit = true,
                this.article.id = article.id;
                this.article.article_id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;
            },
            clear(){
                    this.article.title = '';
                    this.article.body = '';
                
            }
            
        }
    }
</script>