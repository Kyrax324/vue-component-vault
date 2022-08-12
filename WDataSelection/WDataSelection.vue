// v1.0.0
<template>
	<div>
		<slot
			name="activator"
			:on="()=>{
				visible = true
			}"
		></slot>

		<!-- dialog -->
		<v-dialog
			v-model="visible"
			:max-width="maxWidth"
			persistent
		>
			<v-card>
				<v-card-title>
					<v-row>
						<v-col>{{ label }}</v-col>
						<v-col cols="auto">
							<v-btn icon @click="visible = false">
								<v-icon>mdi-close</v-icon>
							</v-btn>
						</v-col>
					</v-row>
				</v-card-title>
				<v-divider></v-divider>
				<v-card-text class="pa-0">
					<div class="pa-2">
						<v-text-field
							v-model="search"
							:placeholder="placeholder"
							prepend-inner-icon="mdi-magnify"
							outlined
							rounded
							dense
							hide-details
							@input="onInput()"
							@keyup.enter="onEnter()"
						></v-text-field>
					</div>
					<div
						v-show="dataObject.data.length > 0"
						class="caption px-6 font-italic text-right pb-1"
					>
						Displaying {{ dataObject.from }} - {{ dataObject.to }} of {{ dataObject.total }}
					</div>
					<div style="height: 45vh;" class="overflow-auto">
						<v-container
							v-if="loading"
							class="fill-height justify-center"
						>
							<v-progress-circular
								indeterminate
								size="80"
								width="5"
								color="primary"
							></v-progress-circular>
						</v-container>
						<v-container
							v-else-if="dataObject.data.length == 0"
							class="fill-height justify-center"
						>
							<div class="grey--text">No Available Results</div>
						</v-container>
						<div v-else>
							<v-list dense>
								<template v-for="(item,i) in dataObject.data">
									<slot
										name="list-item"
										:item="item"
										:active="isActive(item)"
										:onSelect="()=> { onSelect(item) }"
									>
										<v-list-item
											:key="i"
											:input-value="isActive(item)"
											@click="onSelect(item)"
										>
											<v-list-item-content>{{ item[itemText] }}</v-list-item-content>
											<v-list-item-action v-if="multiple">
												<v-icon>{{ isActive(item) ? 'mdi-checkbox-marked' : 'mdi-checkbox-blank-outline' }}</v-icon>
											</v-list-item-action>
										</v-list-item>
									</slot>
								</template>
							</v-list>
						</div>
					</div>
				</v-card-text>
				<v-card-actions>
					<v-row
						no-gutters
						:justify="multiple ? 'space-between' : 'center'"
						align="center"
					>
						<v-col cols="auto">
							<v-pagination
								v-model="page"
								circle
								:length="Math.ceil(dataObject.total / itemsPerPage)"
								:total-visible="0"
								@input="onEnter()"
							></v-pagination>
						</v-col>
						<v-col v-if="multiple" cols="auto">
							<v-btn
								small
								color="primary"
								@click="emitInput()"
							>
								Save
							</v-btn>
						</v-col>
					</v-row>
				</v-card-actions>
			</v-card>
		</v-dialog>
	</div>
</template>

<script>
import cloneDeep from 'lodash/cloneDeep'
export default{
	props:{
		value:{
			type: [Array, Object],
			default: undefined
		},
		multiple:{
			type: Boolean,
			default: false,
		},
		itemText:{
			type: String,
			default: "name",
		},
		itemValue:{
			type: String,
			default: "id",
		},
		label:{
			type: String,
			default: null,
		},
		placeholder:{
			type: String,
			default: null,
		},
		dataObject:{
			type: Object,
			default: () => {
				return {
					"data" : [],
					"total" : 0,
					"from" : 0,
					"to" : 0,
				}
			},
		},
		loading:{
			type: Boolean,
			default: false,
		},
		itemsPerPage:{
			type: Number,
			default: 10,
		},
		delaySeconds:{
			type: Number,
			default: 1 * 1000
		},
		maxWidth: {
			type: [Number, String],
			default: "450"
		},
	},
	data(){
		return {
			search: null,
			visible: false,
			value__ : null,
			timer : null,
			page: 1,
		}
	},
	watch:{
		'value':{
			handler(){
				this.value__ = this.value
			},
			immediate: true,
		},
	},
	created(){
		this.searchData()
	},
	methods:{
		onInput(){
			this.searchWithTimer()
		},
		onEnter(){
			this.searchData()
		},
		searchWithTimer(){
			if (this.timer) {
				clearTimeout(this.timer);
				this.timer = null;
			}
			this.timer = setTimeout(() => {
				this.searchData()
			}, this.delaySeconds);
		},
		searchData(){
			let payload = {
				search: this.search,
				itemsPerPage: this.itemsPerPage,
				page: this.page,
			}
			this.$emit("search", payload)
		},
		isActive(listItem){
			if(this.value__ == null) return false;

			if(this.multiple){
				return (this.value__ ?? []).some((item)=> item[this.itemValue] == listItem[this.itemValue] )
			}else{
				return this.value__[this.itemValue] == listItem[this.itemValue]
			}
		},
		onSelect(listItem){
			if(this.multiple){
				let values = cloneDeep(this.value__) ?? []
				let index = values.findIndex((item)=> item[this.itemValue] == listItem[this.itemValue] )
				if(index >= 0){
					values.splice(index, 1)
				}else{
					values.push(listItem)
				}
				this.value__ = values
			}else{
				this.value__ = listItem
				this.emitInput()
			}
		},
		emitInput(){
			this.$emit("input", this.value__)
			this.visible = false
		},
	}
}
</script>