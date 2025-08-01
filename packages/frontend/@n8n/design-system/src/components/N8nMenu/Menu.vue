<script lang="ts" setup>
import { ElMenu } from 'element-plus';
import { computed, onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';

import type { IMenuItem, IMenuElement } from '../../types';
import { isCustomMenuItem } from '../../types';
import N8nMenuItem from '../N8nMenuItem';
import { doesMenuItemMatchCurrentRoute } from '../N8nMenuItem/routerUtil';

interface MenuProps {
	type?: 'primary' | 'secondary';
	defaultActive?: string;
	collapsed?: boolean;
	transparentBackground?: boolean;
	mode?: 'router' | 'tabs';
	tooltipDelay?: number;
	items?: IMenuElement[];
	modelValue?: string;
}

const props = withDefaults(defineProps<MenuProps>(), {
	type: 'primary',
	collapsed: false,
	transparentBackground: false,
	mode: 'router',
	tooltipDelay: 300,
	items: () => [],
});
const $route = useRoute();

const emit = defineEmits<{
	select: [itemId: string];
	'update:modelValue': [itemId: string];
}>();

const activeTab = ref(props.modelValue);

const upperMenuItems = computed(() =>
	props.items.filter((item: IMenuElement) => item.position === 'top' && item.available !== false),
);

const lowerMenuItems = computed(() =>
	props.items.filter(
		(item: IMenuElement) => item.position === 'bottom' && item.available !== false,
	),
);

const currentRoute = computed(() => {
	return $route ?? { name: '', path: '' };
});

onMounted(() => {
	if (props.mode === 'router') {
		const found = props.items.find((item) =>
			doesMenuItemMatchCurrentRoute(item, currentRoute.value),
		);

		activeTab.value = found ? found.id : '';
	} else {
		activeTab.value = props.items.length > 0 ? props.items[0].id : '';
	}

	emit('update:modelValue', activeTab.value);
});

const onSelect = (item: IMenuItem): void => {
	if (props.mode === 'tabs') {
		activeTab.value = item.id;
	}

	emit('select', item.id);
	emit('update:modelValue', item.id);
};
</script>

<template>
	<div
		:class="{
			['menu-container']: true,
			[$style.container]: true,
			[$style.menuCollapsed]: collapsed,
			[$style.transparentBackground]: transparentBackground,
		}"
	>
		<div v-if="$slots.header" :class="$style.menuHeader">
			<slot name="header"></slot>
		</div>
		<div :class="$style.menuContent">
			<div :class="{ [$style.upperContent]: true, ['pt-xs']: $slots.menuPrefix }">
				<div v-if="$slots.menuPrefix" :class="$style.menuPrefix">
					<slot name="menuPrefix"></slot>
				</div>
				<ElMenu :default-active="defaultActive" :collapse="collapsed">
					<template v-for="item in upperMenuItems" :key="item.id">
						<component :is="item.component" v-if="isCustomMenuItem(item)" v-bind="item.props" />
						<N8nMenuItem
							v-else
							:item="item"
							:compact="collapsed"
							:tooltip-delay="tooltipDelay"
							:mode="mode"
							:active-tab="activeTab"
							:handle-select="onSelect"
						/>
					</template>
				</ElMenu>
			</div>
			<div :class="[$style.lowerContent, 'pb-2xs']">
				<slot name="beforeLowerMenu"></slot>
				<ElMenu :default-active="defaultActive" :collapse="collapsed">
					<template v-for="item in lowerMenuItems" :key="item.id">
						<component :is="item.component" v-if="isCustomMenuItem(item)" v-bind="item.props" />
						<N8nMenuItem
							v-else
							:item="item"
							:compact="collapsed"
							:tooltip-delay="tooltipDelay"
							:mode="mode"
							:active-tab="activeTab"
							:handle-select="onSelect"
						/>
					</template>
				</ElMenu>
				<div v-if="$slots.menuSuffix" :class="$style.menuSuffix">
					<slot name="menuSuffix"></slot>
				</div>
			</div>
		</div>
		<div v-if="$slots.footer" :class="$style.menuFooter">
			<slot name="footer"></slot>
		</div>
	</div>
</template>

<style lang="scss" module>
.container {
	height: 100%;
	display: flex;
	flex-direction: column;
	background-color: var(--menu-background, var(--color-background-xlight));
	overflow: hidden;
}

.menuHeader {
	display: flex;
	flex-direction: column;
	flex: 0 1 auto;
	overflow-y: auto;
}

.menuContent {
	display: flex;
	flex-direction: column;
	justify-content: space-between;
	flex: 1 1 auto;

	& > div > :global(.el-menu) {
		background: none;
		padding: var(--menu-padding, 12px);
	}
}

.upperContent {
	ul {
		padding-top: 0 !important;
	}
}

.lowerContent {
	ul {
		padding-bottom: 0 !important;
	}
}

.menuCollapsed {
	transition: width 150ms ease-in-out;
	:global(.hideme) {
		display: none !important;
	}
}

.transparentBackground {
	background-color: transparent;
}
</style>
