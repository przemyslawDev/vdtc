<template>
    <div style="font-size: 12px;">
        <table class="table table-bordered">
            <thead>
            <tr>
                <th :colspan="columns.length">
					<slot :name="'table-header'" />
					<a v-if="settings.clearableFilters"
						class="text-danger"
						@click="clearFilters()">
						Clear all filters
					</a>
                    <label class="pull-right">
                        Show per page
						<template v-if="settings.perPageOptions">
							<select @change="perPageChange($event.target.value)">
								<option v-for="(option, index) in settings.perPageOptions" :key="index" :value="option">{{ option }}</option>
							</select>
						</template>
						<template v-else>
							<select @change="perPageChange($event.target.value)">
                            	<option v-for="(option, index) in perPageDefaultOptions" :key="index" :value="option">{{ option }}</option>
                        	</select>
						</template>
                    </label>
                </th>
            </tr>
            <tr>
                <th v-for="column in columns" :key="column.name" :id="'header-' + column.name"
                    :style="filteredStyle(column)"
                >
                    <div>
						<slot :name="'clear-filter-' + column.name | lowerCase">
							<i v-if="isFiltered(column) && column.filterableSettings && column.filterableSettings.clearable" 
							class="clear-filter fa fa-times text-danger" 
							@click="clearFilter(column)"></i>
						</slot>
                        {{ column.label }}
                        <template v-if="column.sortable">
                            <div @click="sortColumn(column.name)" style="display: inline-block; float:right;">
                                <span v-if="sort[column.name] === 'asc'"
                                      class="fa fa-sort-asc text-danger"
                                      aria-hidden="true"></span>
                                <span v-else-if="sort[column.name] === 'desc'"
                                      class="fa fa-sort-desc text-danger"
                                      aria-hidden="true"></span>
                                <span v-else
                                      class="fa fa-sort"
                                      aria-hidden="true"></span>
                            </div>
                        </template>
                    </div>
                    <div v-if="column.filterable">
                        <slot :name="'filter-' + column.name | lowerCase"
                              :columnName="column.name"
                              :settings="column.filterableSettings ? column.filterableSettings : null"
                        >
                            <template v-if="column.filterableSettings">
                                <template v-if="column.filterableSettings.type === 'select'">
                                    <select @change="filterSelectChange($event.target.value, column.name)"
                                            :value="filter[column.name]
                                                        ? filter[column.name]
                                                        : (column.filterableSettings.defaultValue
                                                            ? column.filterableSettings.defaultValue
                                                            : column.filterableSettings.options[0].value)"
                                    >
										<template v-for="(option, index) in column.filterableSettings.options">
											<option v-if="option.label && option.value" :value="option.value" :key="index">
												 {{ option.label }}
											</option>
											<option v-else :value="option" :key="index">
												{{ option }}
											</option>
										</template>
                                    </select>
                                </template>
                                <template v-else-if="column.filterableSettings.type === 'input'">
                                    <input @keyup="filterInputKeyup($event.target.value, column.name)"
                                           @keyup.enter="filterInputKeyUpEnter($event.target.value, column.name)"
                                           :value="filter[column.name] ? filter[column.name] :
											(column.filterableSettings.defaultValue ? column.filterableSettings.defaultValue : '')"
                                           class="filter-input"
                                           style="width:  100%;"
                                           :placeholder="column.filterableSettings.placeholder ? column.filterableSettings.placeholder : ''"
                                    >
                                </template>
                            </template>
                        </slot>
                    </div>
                    <div v-else style="height: 24px;">

                    </div>
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(item, index) in data" :key="index">
                <td v-for="column in columns" :key="column.name" :style="column.styleRow ? column.styleRow : null">
                    <slot v-if="column.value"
                          :name="'col-' + column.name | lowerCase"
                          :value="getValue(item, getIndex(column.value))"
                          :row="item">
                        {{ getValue(item, getIndex(column.value)) }}
                    </slot>
                    <slot v-else
                          :name="'col-' + column.name | lowerCase"
                          :value="''"
                          :row="item">
                    </slot>
                </td>
            </tr>
            </tbody>
        </table>
        <pagination v-if="pagination"
                    :info="pagination"
                    @throw-page="throwPage"
        ></pagination>
    </div>
</template>

<script>
import Pagination from './Pagination.vue';

export default {
	data () {
		return {
			sortTimes: {},
			sort: {},
			filter: {},
			perPage: 0,
			perPageDefaultOptions: [10,15,25,50,100]
		};
	},
	props: {
		data: {
			type: Array,
			required: true,
		},
		columns: {
			type: Array,
			required: true
		},
		pagination: {
			type: Object,
			default: null
		},
		settings: {
			type: Object,
			required: true
		}
	},
	mounted () {
		this.setDefaultFilterValues();
		this.perPage = this.pagination.perPage;
		this.$emit('mounted');
	},
	components: {
		'pagination': Pagination
	},
	methods: {
		perPageChange (value) {
			this.perPage = value;
			this.$emit('per-page', this.perPage);
		},
		filterInputKeyup: _.debounce(function (value, columnName) {
			this.filter[columnName] = value;
			this.$emit('filter', this.filter);
		}, 1000),
        filterInputKeyUpEnter (value, columnName) {
            this.filter[columnName] = value;
            this.$emit('filter', this.filter)
        },
		filterSelectChange (value, columnName) {
			this.filter[columnName] = value;
			this.$emit('filter', this.filter);
		},
		throwPage (value) {
			this.$emit('throw-page', value);
		},
		getIndex (value) {
			let indexes = [value];
			if (value.indexOf('.') > 0) indexes = value.split('.');
			return indexes;
		},
		getValue (object, indexes) {
			let value = null;
			for (let i = 0; i < indexes.length; i++) {
				if (i !== indexes.length - 1) {
					object = object[indexes[i]];
				} else {
					if (object) {
						value = object[indexes[i]];
					}
				}
			}
			return value;
		},
		sortColumn (columnName) {
			let sortA = this.sort;
			let sortTimesA = this.sortTimes;
			this.sort = [];

			if (typeof sortTimesA[columnName] === 'undefined' || typeof sortTimesA[columnName] === 'NaN') {
				sortTimesA[columnName] = 0;
			}

			if (typeof sortA[columnName] !== 'undefined') {
				if (sortTimesA[columnName] >= 2) {
					sortTimesA[columnName] = 0;
					sortA[columnName] = '';
				} else {
					sortTimesA[columnName]++;
					if (sortA[columnName] === 'asc') {
						sortA[columnName] = 'desc';
					} else {
						sortA[columnName] = 'asc';
					} 
				}
			} else {
				sortTimesA[columnName]++;
				sortA[columnName] = 'asc';
			}

			this.sort = sortA;
			this.sortTimes = sortTimesA;

			this.$emit('sort', this.sort);
		},
		setDefaultFilterValues () {
			for(let index in this.columns) {
				let column = this.columns[index];
				if(column.filterableSettings && column.filterableSettings.defaultValue) {
					this.filter[column.name] = column.filterableSettings.defaultValue;
					this.$emit('filter', this.filter);
				}
			}
		},
        isFiltered (column) {
		    return this.filter[column.name] && !(this.filter[column.name] === 'all');
		},
        filteredStyle (column) {
			let col_style = {};
			if (column.style) col_style = JSON.parse(JSON.stringify(column)).style;
            return this.isFiltered(column)
                ? Object.assign(col_style, this.settings.filteredStyle)
                : column.style;
        },
        clearFilter (column) {
			this.filter[column.name] = '';
			if (column.filterableSettings) column.filterableSettings.defaultValue = '';
			this.$emit('clear-filter', column.name);
            this.$emit('filter', this.filter);
		},
		clearFilters () {
			for (let index in this.columns) {
				let column = this.columns[index];
				if (column.filterable) this.clearFilter(column);
			}
		}
	},
	filters: {
		lowerCase (value) {
			if (value) return value.toLowerCase();
		}
	}
};
</script>
<style>
    a.vue-table {
        color: inherit;
        text-decoration: underline;
    }
</style>