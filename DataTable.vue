<template>
    <div class="table-responsive">
        <table class="table table-bordered">
            <thead>
                <tr>
                    <th v-for="column in columns">
                        <div>
                            {{ column.label }}
                            <template v-if="column.sortable">
                                <a @click="sortColumn(column.label)">
                                    <span v-if="sort[column.label] === 'asc'" class="glyphicon glyphicon-sort-by-attributes" aria-hidden="true"></span>
                                    <span v-else-if="sort[column.label] === 'desc'" class="glyphicon glyphicon-sort-by-attributes-alt" aria-hidden="true"></span>
                                    <span v-else class="glyphicon glyphicon-sort" aria-hidden="true"></span>
                                </a>
                            </template>
                        </div>
                        <div v-if="column.filterable">
                            <input @keyup="filterInputKeyup($event.target.value, column.label)" class="filter-input" style="width:  100%;" placeholder="Search">
                        </div>
                        <div v-else style="height: 26px;">

                        </div>
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in data">
                    <td v-for="column in columns">
                        <slot :name="'col-' + column.label | lowerCase"
                              :value="getValue(item, getIndex(column.value))"
                              :row="item">
                            {{ getValue(item, getIndex(column.value)) }}
                        </slot>
                    </td>
                </tr>
            </tbody>
            <pagination v-if="pagination" :info="pagination" @currentPage="changePage"></pagination>
        </table>
    </div>
</template>

<script>
    import Pagination from './Pagination.vue';

    export default {
        data() {
            return {
                sort: {},
                filter: {}
            }
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
            }
        },
        components: {
            'pagination': Pagination
        },
        methods: {
            filterInputKeyup: _.debounce(function (value, columnLabel) {
                this.filter[columnLabel] = value;
                this.$emit('filter', this.filter);
            }, 500),
            changePage(val) {
                this.$emit('changePage', val);
            },
            getIndex(val) {
                let indexes = [val];
                if(val.indexOf('.') > 0) {
                    indexes = val.split('.');
                }
                return indexes;
            },
            getValue(object, indexes) {
                let val = null;
                for(let i = 0; i < indexes.length; i++) {
                    if(i !== indexes.length - 1) {
                        object = object[indexes[i]];
                    } else {
                        if(object) {
                            val = object[indexes[i]];
                        }
                    }
                }
                return val;
            },
            sortColumn(columnLabel) {
                let sort_a = this.sort;
                this.sort = [];
                if(typeof sort_a[columnLabel] !== 'undefined') {
                    if(sort_a[columnLabel] === 'asc') {
                        sort_a[columnLabel] = 'desc';
                    } else {
                        sort_a[columnLabel] = 'asc';
                    }
                } else {
                    sort_a[columnLabel] = 'asc';
                }
                this.sort = sort_a;
                this.$emit('sort', this.sort);
            }
        },
        filters: {
            lowerCase(val) {
                if(val) {
                    return val.toLowerCase();
                }
            }
        }
    }
</script>