<template>
	<div class="editor-page">
		<div class="container page">
			<div class="row">
				<div class="col-md-10 offset-md-1 col-xs-12">
					<form>
						<fieldset>
							<fieldset class="form-group">
								<input
									type="text"
									class="form-control form-control-lg"
									placeholder="Article Title"
									v-model="form.title"
								/>
							</fieldset>
							<fieldset class="form-group">
								<input
									type="text"
									class="form-control"
									placeholder="What's this article about?"
									v-model="form.description"
								/>
							</fieldset>
							<fieldset class="form-group">
								<textarea
									class="form-control"
									rows="8"
									placeholder="Write your article (in markdown)"
									v-model="form.body"
								></textarea>
							</fieldset>
							<fieldset class="form-group">
								<input
									type="text"
									class="form-control"
									placeholder="Enter tags"
									v-model="inputTagValue"
									@keyup.enter="addTag"
								/>
								<div class="tag-list">
									<span
										class="tag-default tag-pill ng-binding ng-scope"
										v-for="(tag, index) in form.tagList"
										:key="index"
									>
										<i class="ion-close-round" @click="deleteTag(index)"></i>
										{{ tag }}
									</span>
								</div>
							</fieldset>
							<button class="btn btn-lg pull-xs-right btn-primary" type="button" @click="commit">
								Publish Article
							</button>
						</fieldset>
					</form>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import { createArticle, getArticle, updateArticle } from "@/api/article";
export default {
	// 在路由匹配组件渲染之前会先执行中间件处理
	middleware: "authenticated",
	name: "EditorIndex",
	async asyncData({ params }) {
    const slug = params.slug || null;
    let responesData = {}
    // 获取文章ID
    if(slug) {
      const {
        data: { article },
      } = await getArticle(slug);
      responesData = article
    }
    
		return {
			form: {
        title: "", // 标题
        description: "", // 说明
        body: "", // 文章内容
        tagList: [], // 标签集合
        ...responesData,
      },
			slug,
		};
	},
	data() {
		return {
			inputTagValue: "",
		};
	},
	methods: {
		addTag() {
			this.form.tagList.push(this.inputTagValue);
			this.inputTagValue = "";
		},
		deleteTag(index) {
			this.form.tagList.splice(index, 1);
		},
		async commit() {
			// 提交文章
			if (!this.form.title || !this.form.description || !this.form.body) {
				alert("Please enter a title, description and body");
			} else {
				try {
					let responseData = {};
					if (this.slug) {
						const {
							data: { article },
						} = await updateArticle(this.slug, { article: this.form });
						responseData = article;
					} else {
						const {
							data: { article },
						} = await createArticle({ article: this.form });
						responseData = article;
					}
					this.$router.push({ path: `/article/${responseData.slug}` });
				} catch (error) {
					console.error(error);
				}
			}
		},
	},
};
</script>

<style></style>
