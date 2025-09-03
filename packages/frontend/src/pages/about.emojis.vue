<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div class="_gaps">
	<MkButton v-if="$i && ($i.isModerator || $i.policies.canManageCustomEmojis)" primary link to="/custom-emojis-manager">{{ i18n.ts.manageCustomEmojis }}</MkButton>

	<FormSection>
		<div class="_gaps_m">
			<MkKeyValue>
				<template #key>LICENSE</template>
				<template #value>
					<div>
						Some custom emojis that have the Apache License, Version 2.0 specified in the license field are licensed under the Apache License, Version 2.0.<br><br>
						Copyright 2021 cryoca<br><br>
						Licensed under the Apache License, Version 2.0 (the "License");<br>
						you may not use this file except in compliance with the License.<br>
						You may obtain a copy of the License at<br><br>
						    http://www.apache.org/licenses/LICENSE-2.0<br><br>
						Unless required by applicable law or agreed to in writing, software<br>
						distributed under the License is distributed on an "AS IS" BASIS,<br>
						WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.<br>
						See the License for the specific language governing permissions and<br>
						limitations under the License.
					</div>
				</template>
			</MkKeyValue>
			<FormLink to="https://cryo.jp/misskey/onsen/LICENSE-2.0.txt" external>Apache License, Version 2.0</FormLink>
		</div>
	</FormSection>

	<div class="query">
		<MkInput v-model="q" class="" :placeholder="i18n.ts.search" autocapitalize="off">
			<template #prefix><i class="ti ti-search"></i></template>
		</MkInput>

		<!-- たくさんあると邪魔
		<div class="tags">
			<span class="tag _button" v-for="tag in customEmojiTags" :class="{ active: selectedTags.has(tag) }" @click="toggleTag(tag)">{{ tag }}</span>
		</div>
		-->
	</div>

	<MkFoldableSection v-if="searchEmojis">
		<template #header>{{ i18n.ts.searchResult }}</template>
		<div :class="$style.emojis">
			<XEmoji v-for="emoji in searchEmojis" :key="emoji.name" :emoji="emoji"/>
		</div>
	</MkFoldableSection>

	<MkFoldableSection v-for="category in customEmojiCategories" v-once :key="category ?? '___root___'">
		<template #header>{{ category || i18n.ts.other }}</template>
		<div :class="$style.emojis">
			<XEmoji v-for="emoji in customEmojis.filter(e => e.category === category)" :key="emoji.name" :emoji="emoji"/>
		</div>
	</MkFoldableSection>
</div>
</template>

<script lang="ts" setup>
import { watch, ref } from 'vue';
import * as Misskey from 'misskey-js';
import XEmoji from './emojis.emoji.vue';
import MkButton from '@/components/MkButton.vue';
import MkInput from '@/components/MkInput.vue';
import MkFoldableSection from '@/components/MkFoldableSection.vue';
import MkKeyValue from '@/components/MkKeyValue.vue';
import FormLink from '@/components/form/link.vue';
import { customEmojis, customEmojiCategories, getCustomEmojiTags } from '@/custom-emojis.js';
import { i18n } from '@/i18n.js';
import { $i } from '@/i.js';

const customEmojiTags = getCustomEmojiTags();
const q = ref('');
const searchEmojis = ref<Misskey.entities.EmojiSimple[] | null>(null);
const selectedTags = ref(new Set());

function search() {
	if ((q.value === '' || q.value == null) && selectedTags.value.size === 0) {
		searchEmojis.value = null;
		return;
	}

	if (selectedTags.value.size === 0) {
		const queryarry = q.value.match(/\:([a-z0-9_]*)\:/g);

		if (queryarry) {
			searchEmojis.value = customEmojis.value.filter(emoji =>
				queryarry.includes(`:${emoji.name}:`),
			);
		} else {
			searchEmojis.value = customEmojis.value.filter(emoji => emoji.name.includes(q.value) || emoji.aliases.includes(q.value));
		}
	} else {
		searchEmojis.value = customEmojis.value.filter(emoji => (emoji.name.includes(q.value) || emoji.aliases.includes(q.value)) && [...selectedTags.value].every(t => emoji.aliases.includes(t)));
	}
}

function toggleTag(tag) {
	if (selectedTags.value.has(tag)) {
		selectedTags.value.delete(tag);
	} else {
		selectedTags.value.add(tag);
	}
}

watch(q, () => {
	search();
});

watch(selectedTags, () => {
	search();
}, { deep: true });
</script>

<style lang="scss" module>
.emojis {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(190px, 1fr));
	grid-gap: 12px;
}
</style>
