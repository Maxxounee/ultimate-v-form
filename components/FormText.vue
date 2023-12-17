<template>
	<textarea
		v-model="value"
		class="FormText"
		:placeholder="placeholder"
		@input="(event) => {
			$emit('update', event.target.value);
			trimSpaces(event);
		}"
	></textarea>
</template>

<script>

export default {
	props: {
		name: {
			type: String,
			default: '',
			required: true,
		},
		placeholder: {
			type: String,
			default: '',
		},
		debounce: {
			type: [String, Number],
			default: 300,
		},
		minimalLengthRequired: {
			type: [String, Number],
			default: 0,
		},
	},
	emits: ['update'],
	data() {
		return {
			value: '',
			timeout: undefined,
		};
	},
	watch: {
		value(data) {
			if (this.$parent.setFormValue) {
				const options = {
					name: this.name,
					value: data,
					completelyFilled: this.checkCompletelyFilled(data),
				};
				const call = this.$parent.setFormValue.bind(this, options);
				clearTimeout(this.timeout);
				this.timeout = setTimeout(call, this.debounce);
			}
		},
	},
	mounted() {
		if (this.$parent.setNewField) {
			this.$parent.setNewField(this.name, this.minimalLengthRequired > 0);
		}
	},
	methods: {
		checkCompletelyFilled(value) {
			let available = true;
			if (this.minimalLengthRequired > 0) {
				available = value.length >= this.minimalLengthRequired;
			}
			return available;
		},
		clearValue() {
			this.value = '';
		},
		trimSpaces({ target }) {
			this.value = target.value.replace(/\s+/g, ' ').trimStart();
		},
	},
};
</script>

<style lang="scss">
.FormText {
	/* reset */
	resize: none;

	box-sizing: border-box;
	width: 100%;
	padding: 0;

	font-family: inherit;
	font-weight: inherit;
	color: inherit;
	letter-spacing: inherit;

	background: none;
	border: none;
	border-radius: 0;
	outline: none;

	&::placeholder {
		line-height: inherit;
		color: inherit;
	}

	&:focus::placeholder {
		color: transparent;
	}
}
</style>
