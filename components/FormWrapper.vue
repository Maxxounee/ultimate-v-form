<template>
	<div
		class="FormWrapper"
		:class="{ available, sendPending, resultShown: result }"
	>
		<div class="FormWrapper__form">
			<slot></slot>
		</div>
		<div
			class="FormWrapper__result"
			v-html="result === 'error' ? errorMessage : successMessage"
		></div>
		<div class="FormWrapper__preloader">
			<slot name="preloader">
				<img src="/assets/preloader.svg" />
			</slot>
		</div>
	</div>
</template>

<script>
export default {
	/* TODO: не забыть про возможность работы со счетчиками */
	props: {
		type: {
			type: String,
			default: 'callback',
		},
		additional: {
			type: Object,
			default: () => ({}),
		},
		successMessage: {
			type: String,
			default: 'Заявка отправлена, мы свяжемся с вами в ближайшее время',
		},
		errorMessage: {
			type: String,
			default: 'Произошла ошибка, пожалуйста, попробуйте позже',
		},
		screenTimeout: {
			type: [String, Number],
			default: 3000,
		},
	},
	emits: ['available', 'beforeSend', 'sendSuccess', 'sendError'],
	data() {
		return {
			available: false,
			formValue: {},
			agreeActive: true,
			result: undefined,
			sendPending: false,
		};
	},
	computed: {},
	watch: {
		formValue: {
			handler(data) {
				this.checkAvailable(data);
			},
			deep: true,
		},
		agreeActive() {
			this.checkAvailable(this.formValue);
		},
		available(value) {
			this.$emit('available', value);
		},
	},
	mounted() {
	},
	methods: {
		createFormData() {
			const form = new FormData();
			form.append('type', this.type);
			Object.entries(this.formValue).forEach(([key, value]) => {
				form.append(key, value);
			});
			Object.entries(this.additional).forEach(([key, value]) => {
				form.append(key, value);
			});
			const json = Object.fromEntries(form.entries());
			return { form, json };
		},
		resultHandler(status, json, response) {
			this.sendPending = false;
			this.result = status;
			if (status === 'success') {
				this.$emit('sendSuccess', json, response);
				this.$nuxt.$emit('sendSuccess', json, response);
				this.clearAllFields();
			} else {
				this.$emit('sendError', json, response);
				this.$nuxt.$emit('sendError', json, response);
			}

			setTimeout(() => {
				this.result = undefined;
			}, this.screenTimeout);
		},
		send() {
			if (this.available) {
				this.sendPending = true;
				const { form, json } = this.createFormData();
				this.$emit('beforeSend', json);
				this.$nuxt.$emit('beforeSend', json);
				this.$axios({
					method: 'post',
					url: '/api/feedback',
					data: form,
					headers: { 'Content-Type': 'multipart/form-data' },
				})
					.then((response) => {
						this.resultHandler(response.data.status, json, response);
					})
					.catch((response) => {
						this.resultHandler('error', json, response);
					});
			}
		},
		clearAllFields() {
			this.$slots.default.forEach((node) => {
				if (node?.componentInstance?.clearValue) {
					node.componentInstance.clearValue();
				}
			});
		},
		checkAvailable(data) {
			let available = this.agreeActive;
			for (const key in data) {
				if (!available) {
					break;
				}
				available = !data[key].required || (data[key].required && (data[key].completelyFilled));
			}
			this.available = available;
		},
		setNewField(name, required = false) {
			this.$set(this.formValue, name, {
				value: '',
				completelyFilled: false,
				required,
			});
		},
		setAgreeState(boolean) {
			this.agreeActive = boolean;
		},
		setFormValue({ name, value, completelyFilled = false }) {
			this.$set(this.formValue, name, {
				...this.formValue[name],
				value,
				completelyFilled,
			});
		},
	},
};
</script>

<style lang="scss">
.FormWrapper {
	position: relative;

	&__preloader {
		pointer-events: none;

		position: absolute;
		top: 0;
		left: 0;

		width: 100%;
		height: 100%;

		opacity: 0;

		transition: opacity 0.2s;
	}

	&__form {
		opacity: 1;
		transition: opacity 0.2s;
	}

	&__result {
		pointer-events: none;

		position: absolute;
		top: 0;
		left: 0;

		display: flex;
		align-items: center;
		justify-content: center;

		width: 100%;
		height: 100%;

		text-align: center;

		opacity: 0;

		transition: opacity 0.2s;
	}

	&.sendPending {
		.FormWrapper__preloader {
			opacity: 1;
		}

		.FormWrapper__form {
			pointer-events: none;
			opacity: 0;
		}
	}

	&.resultShown {
		.FormWrapper__form {
			pointer-events: none;
			opacity: 0;
		}

		.FormWrapper__result {
			pointer-events: auto;
			opacity: 1;
		}
	}
}
</style>
