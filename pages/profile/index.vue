<template>
	<div class="profile-page">
		<!-- 个人信息 -->
		<div class="user-info">
			<div class="container">
				<div class="row">
					<div class="col-xs-12 col-md-10 offset-md-1">
						<img :src="profile.image" class="user-img" />
						<h4>{{ profile.username }}</h4>
						<p>{{ profile.bio }}</p>
						<nuxt-link to="/settings">
							<button class="btn btn-sm btn-outline-secondary action-btn">
								<i class="ion-gear-a"></i>
								Edit Profile Settings
							</button>
						</nuxt-link>
					</div>
				</div>
			</div>
		</div>

		<!-- 列表页签 -->
		<div class="container">
			<div class="row">
				<div class="col-xs-12 col-md-10 offset-md-1">
					<div class="list-toggle">
						<ul class="nav nav-pills outline-active">
							<li class="nav-item tabs-item">
								<a :href="`${profile.username}?tab=author`" class="nav-link" :class="{ active: tab === 'author' }">
									My Articles
								</a>
							</li>
							<li class="nav-item tabs-item">
								<a
									:href="`${profile.username}?tab=favorited`"
									class="nav-link"
									:class="{ active: tab === 'favorited' }"
								>
									Favorited Articles
								</a>
							</li>
						</ul>
					</div>

					<!-- 列表 -->
					<template v-if="articles && articles.length">
						<div class="article-preview" v-for="(item, index) in articles" :key="index">
							<div class="article-meta">
								<img :src="item.author && item.author.image" />
								<div class="info">
									<span href="" class="author">{{ item.author.username }}</span>
									<span class="date">{{ item.author.createdAt | date("MMM DD, YYYY") }}</span>
								</div>
								<button
									class="btn btn-primary btn-sm pull-xs-right"
									:class="{ 'btn-outline-primary': !item.favorited }"
									@click="onFavorite(item)"
									:disabled="item.favoriteDisabled"
								>
									<i class="ion-heart"></i> {{ item.favoritesCount }}
								</button>
							</div>
							<nuxt-link :to="`/article/${item.slug}`" class="preview-link">
								<h1>{{ item.title }}</h1>
								<p>{{ item.description }}</p>
								<span>Read more...</span>
								<ul class="tag-articles">
									<template v-for="(tag, ikey) in item.tagList">
										<li class="tag-default tag-pill tag-outline" :key="ikey">{{ tag }}</li>
									</template>
								</ul>
							</nuxt-link>
						</div>
					</template>
					<template v-else>
						<div class="article-preview">No articles are here...</div>
					</template>

					<!-- 分页列表 -->
					<nav v-show="totalPage > 1">
						<ul class="pagination">
							<li
								class="page-item"
								:class="{
									active: item == page,
								}"
								v-for="item in totalPage"
								:key="item"
							>
								<a class="page-link" :href="`${profile.username}?page=${item}&tab=${tab}`">{{ item }}</a>
							</li>
						</ul>
					</nav>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import { getProfiles } from "@/api/user";
import { getArticles, addFavorite, deleteFavorite } from "@/api/article";
export default {
	middleware: "authenticated",
	name: "UserProfile",
	async asyncData({ params, query }) {
		const page = query.page || "1";
		const limit = 5;
		const tab = query.tab || "author";
		const {
			data: { profile },
		} = await getProfiles(params.username);
		const {
			data: { articles, articlesCount },
		} = await getArticles({
			[tab]: params.username,
			limit,
			offset: (page - 1) * limit,
		});
		return {
			page,
			limit,
			tab,
			profile,
			articles,
			articlesCount,
		};
	},
	data() {
		return {};
	},
	computed: {
		totalPage() {
			return Math.ceil(this.articlesCount / this.limit);
		},
	},
	methods: {
		async onFavorite(article) {
			article.favoriteDisabled = true;
			if (article.favorited) {
				// 取消点赞
				await deleteFavorite(article.slug);
				article.favorited = false;
				article.favoritesCount += -1;
			} else {
				// 添加点赞
				await addFavorite(article.slug);
				article.favorited = true;
				article.favoritesCount += 1;
			}
			article.favoriteDisabled = false;
		},
	},
};
</script>

<style>
.tabs-item {
	cursor: pointer;
}
</style>
