<template>
	<div class="article-meta">
		<nuxt-link
			:to="{
				name: 'profile',
				params: {
					username: article.author.username,
				},
			}"
		>
			<img :src="article.author.image" />
		</nuxt-link>
		<div class="info">
			<nuxt-link
				class="author"
				:to="{
					name: 'profile',
					params: {
						username: article.author.username,
					},
				}"
			>
				{{ article.author.username }}
			</nuxt-link>
			<span class="date">{{ article.createdAt | date("MMM DD, YYYY") }}</span>
		</div>

		<template v-if="editableOrNot">
			<button class="btn btn-sm btn-outline-secondary" @click="editArticle()">
				<i class="ion-edit"></i>
				Edit Article
			</button>
			<button class="btn btn-outline-danger btn-sm" @click="deleteArticle()">
				<i class="ion-edit"></i>
				Delete Article
			</button>
		</template>

		<template v-else>
			<button
				class="btn btn-sm btn-outline-secondary"
				:class="{
					active: article.author.following,
				}"
				@click="onFollow(article)"
			>
				<i class="ion-plus-round"></i>
				{{ article.author.following ? "Unfollow" : "Follow" }}
				{{ article.author.username }}
			</button>
			<button
				class="btn btn-sm btn-outline-primary"
				:class="{
					active: article.favorited,
				}"
				@click="onFavorite(article)"
			>
				<i class="ion-heart"></i>
				{{ article.favorited ? "Unfavorite Article" : "Favorite Article" }}
				Favorite Article
				<span class="counter">({{ article.favoritesCount }})</span>
			</button>
		</template>
	</div>
</template>

<script>
import { addFavorite, deleteFavorite, addFollow, deleteFollow } from "@/api/article";
export default {
	name: "ArticleMeta",
	props: {
    editableOrNot: Boolean, // 是否可编辑
		article: {
			type: Object,
			required: true,
		},
  },
	methods: {
		editArticle() {
			this.$router.push({ path: `/editor/${this.article.slug}` });
		},
		async deleteArticle() {
			await deleteArticle(this.article.slug);
			this.$router.push({ name: "home" });
		},
		async onFavorite(article) {
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
		},
		async onFollow(article) {
			if (article.author.following) {
				// 取消关注
				await deleteFollow(article.author.username);
				this.$set(this.article.author, "following", false);
			} else {
				// 添加关注
				await addFollow(article.author.username);
				this.$set(this.article.author, "following", true);
			}
		},
	},
};
</script>

<style></style>
