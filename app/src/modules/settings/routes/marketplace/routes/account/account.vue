<script setup lang="ts">
import api from '@/api';
import VBanner from '@/components/v-banner.vue';
import type { RegistryAccountResponse, RegistryListResponse } from '@directus/extensions-registry';
import { computed, ref, watchEffect } from 'vue';
import { useI18n } from 'vue-i18n';
import { useRouter } from 'vue-router';
import SettingsNavigation from '../../../../components/navigation.vue';
import ExtensionListItem from '../../components/extension-list-item.vue';
import AccountBanner from './components/account-banner.vue';
import AccountInfoSidebarDetail from './components/account-info-sidebar-detail.vue';
import AccountMetadata from './components/account-metadata.vue';

const props = defineProps<{
	accountId: string;
}>();

const router = useRouter();
const { t } = useI18n();

const loading = ref(false);
const error = ref<unknown>(null);
const account = ref<RegistryAccountResponse['data']>();

watchEffect(async () => {
	if (!props.accountId) return;

	loading.value = true;

	try {
		const response = await api.get(`/extensions/registry/account/${props.accountId}`);
		account.value = response.data.data;
	} catch (err) {
		error.value = err;
	} finally {
		loading.value = false;
	}
});

const perPage = 6;
const page = ref(1);
const extensions = ref<RegistryListResponse['data']>([]);
const filterCount = ref(0);
const pageCount = computed(() => Math.round(filterCount.value / perPage));

watchEffect(async () => {
	const { data } = await api.get('/extensions/registry', {
		params: {
			limit: perPage,
			offset: (page.value - 1) * perPage,
			filter: { by: { _eq: props.accountId } },
		},
	});

	filterCount.value = data.meta.filter_count;
	extensions.value = data.data;
});

const navigateBack = () => {
	const backState = router.options.history.state.back;

	if (typeof backState !== 'string' || !backState.startsWith('/login')) {
		router.back();
		return;
	}

	router.push('/settings/marketplace');
};
</script>

<template>
	<private-view :title="t('marketplace')">
		<template #title-outer:prepend>
			<v-button v-tooltip.bottom="t('back')" class="header-icon" rounded icon secondary exact @click="navigateBack">
				<v-icon name="arrow_back" />
			</v-button>
		</template>

		<template #navigation>
			<settings-navigation />
		</template>

		<template #sidebar>
			<account-info-sidebar-detail />
		</template>

		<div class="account-content">
			<template v-if="account">
				<div class="container">
					<div class="grid">
						<AccountBanner class="banner" :account="account" />
						<AccountMetadata class="metadata" :account="account" />
						<v-list>
							<ExtensionListItem v-for="extension in extensions" :key="extension.id" :extension="extension" />
						</v-list>

						<v-pagination
							v-if="pageCount > 1"
							v-model="page"
							class="pagination"
							:length="pageCount"
							:total-visible="5"
						/>
					</div>
				</div>
			</template>

			<v-banner v-else-if="loading" icon="plugin">
				<template #avatar><v-progress-circular indeterminate /></template>
				{{ t('loading') }}
			</v-banner>

			<v-error v-else :error="error" />
		</div>
	</private-view>
</template>

<style scoped lang="scss">
.account-content {
	padding: var(--content-padding);
	padding-block: 0 var(--content-padding-bottom);
	max-inline-size: 1200px;
	inline-size: 100%;
}

.container {
	container-type: inline-size;
	container-name: item;
}

.grid {
	display: grid;
	gap: 40px;
	grid-template-areas: 'banner' 'metadata' 'readme';

	.banner {
		grid-area: banner;
	}

	.readme {
		grid-area: readme;
		min-inline-size: 0;
	}

	.metadata {
		grid-area: metadata;
	}

	@container item (width > 800px) {
		grid-template-columns: 1fr 320px;
		grid-template-areas:
			'banner banner'
			'readme metadata';
	}
}
</style>
