<template>
	<div
		class="FormAgree"
		:class="{ active }"
		@click="active = !active"
	>
		<div class="FormAgree__checkbox">
			<slot name="before">
				<div class="FormAgree__default-checkbox"></div>
			</slot>
		</div>
		<div class="FormAgree__text">
			<slot>Согласие на обработку персональных данных</slot>
		</div>
	</div>
</template>

<script>
export default {
	props: {
		initialActive: {
			type: Boolean,
			default: false,
		},
	},
	emits: ['update'],
	data() {
		return {
			active: false,
		};
	},
	watch: {
		active(value) {
			this.$emit('update', value);
			if (this.$parent.setAgreeState) {
				this.$parent.setAgreeState(value);
			}
		},
	},
	mounted() {
		this.active = this.initialActive;
		if (this.$parent.setAgreeState) {
			this.$parent.setAgreeState(this.active);
		}
	},
	methods: {},
};
</script>

<style lang="scss">
.FormAgree {
	cursor: pointer;
	user-select: none;

	&__default-checkbox {
		width: 10px;
		height: 10px;
		border: 1px solid black;
		border-radius: 0.1rem;
	}

	&.active {
		.FormAgree__default-checkbox {
			background-color: black;
		}
	}
}
</style>
