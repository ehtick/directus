<script setup lang="ts">
import { useSizeClass } from '@directus/composables';

interface Props {
	/** Render as a tile (square) */
	tile?: boolean;
	/** Renders a smaller avatar */
	xSmall?: boolean;
	/** Renders a small avatar */
	small?: boolean;
	/** Renders a large avatar */
	large?: boolean;
	/** Renders a larger avatar */
	xLarge?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
	tile: false,
});

const sizeClass = useSizeClass(props);
</script>

<template>
	<div class="v-avatar" :class="[{ tile }, sizeClass]">
		<slot />
	</div>
</template>

<style scoped>
/*

	Available Variables:

		--v-avatar-color  [var(--theme--background-normal)]
		--v-avatar-size   [48px]

*/

.v-avatar {
	position: relative;
	display: flex;
	align-items: center;
	justify-content: center;
	inline-size: var(--v-avatar-size, 48px);
	block-size: var(--v-avatar-size, 48px);
	overflow: hidden;
	color: var(--theme--foreground-subdued);
	white-space: nowrap;
	text-overflow: ellipsis;
	background-color: var(--v-avatar-color, var(--theme--background-normal));
	border-radius: var(--theme--border-radius);
}

.tile {
	border-radius: 0;
}

.x-small {
	--v-avatar-size: 24px;

	border-radius: 4px;
}

.small {
	--v-avatar-size: 36px;
}

.large {
	--v-avatar-size: 60px;
}

.x-large {
	--v-avatar-size: 80px;
}

:slotted(img) {
	inline-size: 100%;
	block-size: 100%;
	object-fit: cover;
}
</style>
