<template>
	<div
		class="FormSend"
		@click="sendForm"
	>
		<slot></slot>
	</div>
</template>

<script>

export default {
	emits: ['click'],
	data() {
		return {};
	},
	methods: {
		sendForm() {
			this.$emit('click');
			if (this.$parent.send) {
				this.$parent.send();
			}
		},
	},
};
</script>

<style lang="scss">
.FormSend {
}
</style>
