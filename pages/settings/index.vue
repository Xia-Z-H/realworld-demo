<template>
	<div class="settings-page">
		<div class="container page">
			<div class="row">
				<div class="col-md-6 offset-md-3 col-xs-12">
					<h1 class="text-xs-center">Your Settings</h1>

					<form>
						<fieldset>
							<fieldset class="form-group">
								<input class="form-control" type="text" placeholder="URL of profile picture" v-model="userInfo.image" />
							</fieldset>
							<fieldset class="form-group">
								<input
									class="form-control form-control-lg"
									type="text"
									placeholder="Your Name"
									v-model="userInfo.username"
								/>
							</fieldset>
							<fieldset class="form-group">
								<textarea
									class="form-control form-control-lg"
									rows="8"
									placeholder="Short bio about you"
									v-model="userInfo.bio"
								></textarea>
							</fieldset>
							<fieldset class="form-group">
								<input class="form-control form-control-lg" type="text" placeholder="Email" v-model="userInfo.email" />
							</fieldset>
							<fieldset class="form-group">
								<input
									class="form-control form-control-lg"
									type="password"
									placeholder="Password"
									v-model="userInfo.password"
								/>
							</fieldset>
							<button type="button" class="btn btn-lg btn-primary pull-xs-right" @click="updateSetting">
								Update Settings
							</button>
						</fieldset>
					</form>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import { getUser, updateUser } from "@/api/user";
import cookie from "js-cookie";
export default {
	middleware: "authenticated",
	name: "SettingsIndex",
	async asyncData() {
		const { data } = await getUser();
		return {
			userInfo: {
				password: null,
				...data.user,
			},
		};
	},
	methods: {
		async updateSetting() {
			try {
				const {
					data: { user },
				} = await updateUser({
					user: {
						email: this.userInfo.email,
						bio: this.userInfo.bio,
						image: this.userInfo.image,
						password: this.userInfo.password || null,
						username: this.userInfo.username,
					},
				});
				this.$store.commit("setUser", user); // 更新用户信息
				cookie.set("user", user);
				this.$router.push({ path: `/profile/${this.userInfo.username}` });
			} catch (error) {
				console.error(error);
			}
		},
	},
};
</script>

<style></style>
