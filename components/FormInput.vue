<template>
	<input
		v-model="value"
		v-mask="maskDict[mask]"
		class="FormInput"
		:class="{ fake }"
		:placeholder="placeholder"
		:type="inputType"
		@input="(event) => {
			$emit('update', event.target.value);
			trimSpaces(event);
		}"
		@focus="setInitialValue($event, initialValue)"
		@blur="clearBlankValue($event, initialValue)"
	/>
</template>

<script>
import { VueMaskDirective } from 'v-mask';

export default {
	components: {},
	directives: {
		mask: VueMaskDirective,
	},
	props: {
		name: {
			type: String,
			default: 'phone',
			required: true,
		},
		inputType: {
			type: String,
			validator(value) {
				return [
					'tel',
					'email',
					'text',
					'password',
					'url',
					'date',
				].includes(value.trim().toLowerCase());
			},
			required: true,
			default: 'tel',
		},
		/* TODO: допилить фейк поле */
		fake: {
			type: Boolean,
			default: false,
		},
		mask: {
			type: String,
			default: '',
		},
		placeholder: {
			type: String,
			default: '',
			required: true,
		},
		initialValue: {
			type: String,
			default: '',
		},
		passwordLength: {
			type: [Number, String],
			default: 8,
		},
		required: {
			type: Boolean,
			default: false,
		},
		debounce: {
			type: [String, Number],
			default: 140,
		},
	},
	emits: ['update'],
	data() {
		return {
			value: '',
			timeout: undefined,
			maskDict: {
				phone: '+7 (###) ###-##-##',
				phone8: '8 (###) ###-##-##',
			},
			regExp: {
				emailCheck: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
			},
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
			this.$parent.setNewField(this.name, this.required);
		}
	},
	methods: {
		checkCompletelyFilled(value) {
			let available = true;
			if (this.mask) {
				const mask = this.maskDict[this.mask] ?? this.mask;
				available = value.trim().replace(/\d/g, '#') === mask.replace(/\d/g, '#');
			}
			if (this.inputType === 'email') {
				available = this.regExp.emailCheck.test(value);
			}
			if (this.inputType === 'password') {
				available = value.length >= this.passwordLength;
			}
			return available;
		},
		setInitialValue({ target }, value = '') {
			if (this.initialValue && !target.value) {
				target.value = value;
				setTimeout(() => {
					target.setSelectionRange(target.value.length, target.value.length);
				}, 10);
			}
		},
		clearBlankValue({ target }, value = '') {
			if (target.value.length <= value.length) {
				target.value = '';
			}
		},
		clearValue() {
			this.value = '';
		},
		trimSpaces({ target }) {
			if (this.inputType !== 'tel') {
				this.value = target.value.replace(/\s/g, '');
			}
		},
	},
};
</script>

<style lang="scss">
.FormInput {
	/* reset */
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

	&.fake {
		pointer-events: none;
		position: absolute;
		height: 1px;
		opacity: 0;
	}
}
</style>
