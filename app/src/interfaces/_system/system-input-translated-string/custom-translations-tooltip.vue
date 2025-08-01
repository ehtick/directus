<script setup lang="ts">
import type { Translation } from '@/stores/translations';
import { fetchAll } from '@/utils/fetch-all';
import { Ref, ref } from 'vue';
import { useI18n } from 'vue-i18n';

interface Props {
	translationKey: string;
}

const props = defineProps<Props>();

const translations: Ref<Translation[]> = ref([]);
const loading = ref(false);

const { t } = useI18n();

const fetchTranslation = async () => {
	loading.value = true;

	try {
		translations.value = await fetchAll(`/translations`, {
			params: {
				fields: ['language', 'key', 'value'],
				sort: ['language'],
				filter: {
					key: { _eq: props.translationKey },
				},
			},
		});
	} catch {
		// Ignored
	} finally {
		loading.value = false;
	}
};

const clicked = (toggleTooltip: () => void) => {
	toggleTooltip();

	if (!loading.value && translations.value.length === 0) {
		fetchTranslation();
	}
};
</script>

<template>
	<div class="custom-translations-display">
		<v-menu class="menu" show-arrow>
			<template #activator="{ toggle, deactivate, active }">
				<v-icon
					v-tooltip.bottom="translations && translations.length === 0 && t('translations')"
					:small="false"
					class="icon"
					:class="{ active }"
					name="info"
					clickable
					@click.stop="clicked(toggle)"
					@blur="deactivate"
				></v-icon>
			</template>

			<v-list class="translations">
				<v-list-item v-if="translations.length === 0">
					<v-list-item-content>
						<div class="header">
							<div class="lang">
								{{ t('loading') }}
							</div>
						</div>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-for="item in translations" v-else :key="item.language">
					<v-list-item-content>
						<div class="header">
							<div class="lang">
								<v-icon name="translate" small />
								{{ item.language }}
							</div>
						</div>
						<ValueNull v-if="!item.value" />
						<div v-else class="translation-item-text">{{ item.value }}</div>
					</v-list-item-content>
				</v-list-item>
			</v-list>
		</v-menu>
	</div>
</template>

<style lang="scss" scoped>
.v-list {
	inline-size: 300px;
}

.custom-translations-display {
	display: flex;
	align-items: center;

	.icon {
		color: var(--theme--form--field--input--foreground-subdued);
		opacity: 0;
		transition: opacity var(--fast) var(--transition);
	}

	&:hover .icon,
	.icon.active {
		opacity: 1;
	}
}

.translation-display-text {
	margin-inline-end: 4px;
	padding: 2px 0;
}

.translation-item-text {
	padding-block-start: 2px;
}

.translation-display-text,
.translation-item-text {
	overflow: hidden;
	white-space: nowrap;
	text-overflow: ellipsis;
}

.header {
	display: flex;
	gap: 20px;
	align-items: center;
	justify-content: space-between;
	color: var(--theme--form--field--input--foreground-subdued);
	font-size: 12px;

	.lang {
		font-weight: 600;
	}

	.v-icon {
		margin-inline-end: 4px;
	}

	.v-progress-linear {
		flex: 1;
		inline-size: unset;
		max-inline-size: 100px;
		border-radius: 4px;
	}
}

.v-list-item-content {
	padding-block: 4px 2px;
}

.v-list-item:not(:first-child) {
	.header {
		padding-block-start: 8px;
		border-block-start: var(--theme--border-width) solid var(--theme--border-color-subdued);
	}
}
</style>
