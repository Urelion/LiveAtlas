<!--
  - Copyright 2022 James Lyne
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  - http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
	<div v-if="maps.length" class="world">
		<span class="world__name" aria-hidden="true">{{ world.displayName }}</span>
		<div class="world__maps menu">
			<template v-for="map in maps" :key="`${map.world.name}_${map.name}`">
        <input :id="`${name}-${map.world.name}-${map.name}`" type="radio" :name="name"
               v-bind:value="[map.world.name,map.name]" v-model="currentMap"
               :aria-labelledby="`${name}-${map.world.name}-${map.name}-label`">
        <label :id="`${name}-${map.world.name}-${map.name}-label`" class="map"
               :for="`${name}-${map.world.name}-${map.name}`"
               :title="`${map.world.displayName} - ${map.displayName}`">
					<img v-if="map.hasCustomIcon()" :src="map.getIcon()" alt="" />
					<SvgIcon v-else :name="map.getIcon()"></SvgIcon>
				</label>
			</template>
		</div>
	</div>
</template>

<script lang="ts">
import {computed, defineComponent} from 'vue';
import {LiveAtlasWorldDefinition} from "@/index";
import LiveAtlasMapDefinition from "@/model/LiveAtlasMapDefinition";
import {useStore} from "@/store";
import {MutationTypes} from "@/store/mutation-types";
import SvgIcon from "@/components/SvgIcon.vue";
import "@/assets/icons/block_world_surface.svg";
import "@/assets/icons/block_world_cave.svg";
import "@/assets/icons/block_world_biome.svg";
import "@/assets/icons/block_world_flat.svg";
import "@/assets/icons/block_nether_flat.svg";
import "@/assets/icons/block_nether_surface.svg";
import "@/assets/icons/block_the_end_flat.svg";
import "@/assets/icons/block_the_end_surface.svg";
import "@/assets/icons/block_other.svg";
import "@/assets/icons/block_other_flat.svg";
import "@/assets/icons/block_skylands.svg";

export default defineComponent({
	name: 'WorldListItem',
	components: {SvgIcon},
	props: {
		world: {
			type: Object as () => LiveAtlasWorldDefinition,
			required: true
		},
		name: {
			type: String,
			default: 'map',
		}
	},

	setup(props) {
		const store = useStore(),
			maps = computed(() => {
				const maps: LiveAtlasMapDefinition[] = [];

				//Filter out maps appended to other worlds
				props.world.maps.forEach(map => {
					if(!map.appendedWorld || map.appendedWorld.name === props.world.name) {
						maps.push(map);
					}
				});

				return maps;
			}),
			currentMap = computed({
				get: () => store.state.currentMap ? [store.state.currentWorld!.name, store.state.currentMap.name] : undefined,
				set: (value) => value && store.commit(MutationTypes.SET_CURRENT_MAP, {
					worldName: value[0],
					mapName: value[1]
				})
			});

		return {
			currentMap,
			maps
		}
	}
});
</script>

<style lang="scss" scoped>
	.world {
		display: flex;
		align-items: center;
		margin-bottom:  .5rem;
		padding-left: 0.8rem;

		.world__name {
			word-break: break-word;
			overflow-wrap: break-word;
		}

		.world__maps {
			display: flex;
			flex: 0 0 auto;
			flex-wrap: wrap;
			max-width: 11.1rem;
			align-items: center;
			margin-left: auto;
			padding-left: 1rem;
			padding-right: 0.2rem;
			list-style: none;
			margin-right: -0.5rem;
		}
	}

	.map {
		width: 3.2rem;
		height: 3.2rem;
		margin-right: 0.5rem;

		.svg-icon, img {
			position: absolute;
			top: 0.2rem !important;
			right: 0.2rem !important;
			bottom: 0.2rem !important;
			left: 0.2rem !important;
			width: calc(100% - 0.4rem) !important;
			height: auto !important;
		}
	}
</style>
