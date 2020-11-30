<template>
	<div>
		<div class="uk-container">
			<form class="uk-form-stacked">
				<div class="uk-form-row">
					<label class="uk-form-label"
						>Индекс почтового отделения</label
					>
					<div class="uk-form-controls">
						<input
							type="text"
							name="index"
							class="uk-input"
							v-model="zip"
						/>
					</div>
				</div>
				<div class="uk-form-row">
					<label class="uk-form-label">Адрес доставки</label>
					<div class="uk-form-controls">
						<textarea
							name="street"
							class="uk-textarea"
							v-model="street"
						></textarea>
						<div class="highlight" v-if="finded.length">
							<div
								v-for="(item, index) in finded"
								:key="index"
								@click="selectFind(item)"
							>
								{{ item.value }}
							</div>
						</div>
					</div>
				</div>
			</form>
		</div>
	</div>
</template>

<script>
import { debounce } from "lodash";
export default {
	name: "SearchAddress",
	data() {
		return {
			url:
				"https://suggestions.dadata.ru/suggestions/api/4_1/rs/suggest/address",
			token: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", //your token
			zip: "",
			street: "",
			finded: [],
			selected: false,
			words: 0,
		};
	},
	created() {
		this.debouncedGetAnswer = debounce(this.findAddress, 1000);
	},
	mounted() {},
	watch: {
		street(value) {
			const vm = this;
			if (!vm.selected && vm.words > 2) {
				vm.debouncedGetAnswer();
			}
			vm.selected = false;
			vm.words = value.length;
		},
	},
	methods: {
		clearFind() {
			const vm = this;
			vm.finded = [];
			vm.words = 0;
		},
		selectFind(obj) {
			const vm = this;
			vm.selected = true;
			vm.street = obj.value;
			vm.zip = obj.data.postal_code;
			vm.clearFind();
		},
		findAddress() {
			const vm = this;

			var options = {
				method: "POST",
				mode: "cors",
				headers: {
					"Content-Type": "application/json",
					Accept: "application/json",
					Authorization: "Token " + vm.token,
				},
				body: JSON.stringify({ query: vm.street }),
			};

			fetch(vm.url, options)
				.then((response) => response.json())
				.then((result) => {
					vm.finded = result.suggestions;
				})
				.catch((error) => console.error("error", error));
		},
	},
};
</script>
<style scoped>
.highlight {
	border: 1px solid #ccc;
}
.highlight > div {
	padding: 2px 10px;
	cursor: pointer;
}
.highlight > div:hover {
	background: #ddd;
}
</style>