<template>
	<div>
		<h2>Articles</h2>
		<form v-on:submit.prevent="addArticle" class="mb-3">
			<div class="form-group">
				<input type="text" class="form-control" placeholder="title"
					v-model="article.title">
				<textarea class="form-control" placeholder="body"
					v-model="article.body"></textarea>
					<button type="submit" class="btn btn-light btn-block">Save</button>
			</div>
		</form>
		<nav aria-label="Page navigation example">
		  <ul class="pagination">
		    <li v-bind:class="[{disabled:!pagination.prev_page_url}]" class="page-item">
					<a class="page-link" href="#" v-on:click="fetchArticles(pagination.prev_page_url)">
						Previous
					</a>
				</li>
				<li class="page-item disabled">
					<a class="page-link text-dark" href="#">
						Page {{pagination.current_page}} of {{pagination.last_page}}
					</a>
				</li>
		    <li v-bind:class="[{disabled:!pagination.next_page_url}]" class="page-item">
					<a class="page-link" href="#" v-on:click="fetchArticles(pagination.next_page_url)">
						Next
					</a>
				</li>
		  </ul>
		</nav>
		<div class="card card-body mb-2" v-for="a in articles" v-bind:key="a.id">
			<h3>{{a.title}}</h3>
			<p>{{a.body}}</p>
			<hr>
			<button v-on:click="editArticle(a)"
				class="btn btn-warning mb-2">
				Edit
			</button>
			<button v-on:click="deleteArticle(a.id)"
				class="btn btn-danger">
				Delete
			</button>
		</div>
	</div>
</template>

<script>
export default {
	data() {
		return {
				articles: [],
				article: {
					id:'',
					title:'',
					body:''
				},
				pagination:{},
				edit: false,
				test: ' Hi!'
		}
	},
	created: function() {
		this.fetchArticles();
	},

	methods: {
		fetchArticles: function(page_url) {
			page_url = page_url || 'api/article';
			axios.get(page_url).then(res => {
				//console.log(res);
				//console.log(res.data);
				//console.log('links:'+res.data.links);
				res = res.data;
				this.articles = res.data;
				this.makePagination(res.meta, res.links);
			}).catch(err=>console.log(err));
		},
		makePagination: function(meta, links) {
			let pagination = {
				current_page : meta.current_page,
				last_page : meta.last_page,
				next_page_url : links.next,
				prev_page_url : links.prev
			};
			this.pagination = pagination;
		},
		deleteArticle: function(id) {
			if(confirm('Are you sure?')) {
				axios.delete('api/article/'+id).then(res=>{
					alert('Article removed');
					this.fetchArticles();
				});
			}
		},
		addArticle: function() {
			if(this.edit==false) { // add
				axios.post('api/article', {
					title: this.article.title,
					body: this.article.body
				}).then(res=>{
					this.article.title='';
					this.article.body='';
					alert('Article added');
					this.fetchArticles();
				}).catch(err=>console.log(err));
			} else { // update
				axios.put('api/article/'+this.article.id,
					{
						id: this.article.id,
						title:this.article.title,
						body:this.article.body
					}).then(res=>{
						this.article.title = '';
						this.article.body = '';
						alert('Article updated');
						this.fetchArticles();
					}).catch(err=>console.log(err));
					this.edit = false;
			}
		},
		editArticle: function(article) {
			this.article.id = article.id;
			this.article.title = article.title;
			this.article.body = article.body;
			this.edit = true;
		}
	}

}
</script>
