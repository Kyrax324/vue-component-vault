## Example:

Basic Usage

```vue
<template>
	<w-data-selection
		v-model="form.brand"
		label="Select Brand"
		placeholder="Search for Brands"
		:data-object="selection_brand.dataObj"
		:loading="selection_brand.loading"
		@search="searchBrand"
	>
		<template #activator="{ on }">
			<v-text-field
				:value="form.brand ? form.brand.name : null"
				label="Brand" placeholder="Select Brand"
				readonly
				dense
				:disabled="action == 'edit'"
				:error-messages="errors.brand_id"
				@click="on"
			></v-text-field>
		</template>
	</w-data-selection>
</template>


<script>
export default{
	data () {
		return {
			selection_brand: {},
		}
	}

	methods:{
		searchBrand(option){
			let payload = {
				page: option.page,
				itemsPerPage: option.itemsPerPage,
				search: option.search,
				searchBy: "name",
			}
			// fetch api
		},
	}
}
</script>
```

advance usage:

```vue
	<w-data-selection
		v-model="form.categories"
		label="Select Category"
		placeholder="Search for Categories"
		multiple
		:data-object="selection_game_category.dataObj"
		:loading="selection_game_category.loading"
		@search="searchGameCategory"
	>
		<template #activator="{ on }">
			<v-row dense align="center" class="pb-4">
				<v-col cols="auto">Categories:</v-col>
				<v-col cols="auto">
					<v-btn
						icon small
						@click="on"
					>
						<v-icon small>mdi-plus-circle</v-icon>
					</v-btn>
				</v-col>
			</v-row>
			<template v-for="(category,i) in form.categories">
				<v-chip
					:key="i"
					small
					close
					class="mr-1 mb-1"
					@click:close="form.categories.splice(i,1)"
				>
					{{ category.name }}
				</v-chip>
			</template>
		</template>
		<template #list-item="{ item: data, active, onSelect }">
			<v-list-item @click="onSelect">
				<v-list-item-content>{{ data.name }}</v-list-item-content>
				<v-list-item-action>
					<v-icon>{{ active ? 'mdi-checkbox-marked' : 'mdi-checkbox-blank-outline' }}</v-icon>
				</v-list-item-action>
			</v-list-item>
		</template>
	</w-data-selection>
```