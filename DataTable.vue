<template>
    <div>
    <label v-if="settings.perPage">
      Show
      <select @change="perPageChange($event.target.value)">
        <option value="10">10</option>
        <option value="15">15</option>
        <option value="25">25</option>
        <option value="50">50</option>
        <option value="100">100</option>
      </select>
    </label>
    <table class="table table-bordered">
      <thead>
      <tr>
        <th :style="column.style" v-for="column in columns">
          <div>
            {{ column.label }}
            <template v-if="column.sortable">
              <a @click="sortColumn(column.name)">
                                <span v-if="sort[column.name] === 'asc'"
                                      class="glyphicon glyphicon-sort-by-attributes"
                                      aria-hidden="true"></span>
                <span v-else-if="sort[column.name] === 'desc'"
                      class="glyphicon glyphicon-sort-by-attributes-alt"
                      aria-hidden="true"></span>
                <span v-else
                      class="glyphicon glyphicon-sort"
                      aria-hidden="true"></span>
              </a>
            </template>
          </div>
          <div v-if="column.filterable">
            <slot :name="'filter-' + column.name | lowerCase"
                  :columnName="column.name"
                  :settings="column.filterableSettings ? column.filterableSettings : null">
              <template v-if="column.filterableSettings">
                <select v-if="column.filterableSettings.type === 'select'"
                        @change="filterSelectChange($event.target.value, column.name)">
                  <option v-for="option in column.filterableSettings.options"
                          :value="option.value">
                    {{ option.label }}
                  </option>
                </select>
              </template>
              <input v-else @keyup="filterInputKeyup($event.target.value, column.name)"
                     :value="filter[column.name] ? filter[column.name] : (column.filterableSettings && column.filterableSettings.defaultValue ? column.filterableSettings.defaultValue : '')"
                     class="filter-input"
                     style="width:  100%;" placeholder="Search">
            </slot>
          </div>
          <div v-else style="height: 24px;">

          </div>
        </th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="item in data">
        <td v-for="column in columns">
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
                @throw-page="throwPage"></pagination>
  </div>
</template>

<script>
  import Pagination from './Pagination.vue'

  export default {
    data () {
      return {
        sort: {},
        filter: {},
        perPage: 0
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
      },
      settings: {
        type: Object,
        required: true
      }
    },
    mounted () {
      this.setDefaultFilterValues()
      this.perPage = this.pagination.perPage
    },
    components: {
      'pagination': Pagination
    },
    methods: {
      perPageChange (value) {
        this.perPage = value
        this.$emit('per-page', this.perPage)
      },
      filterInputKeyup: _.debounce(function (value, columnName) {
        this.filter[columnName] = value
        this.$emit('filter', this.filter)
      }, 2000),
      filterSelectChange (value, columnName) {
        this.filter[columnName] = value
        this.$emit('filter', this.filter)
      },
      throwPage (value) {
        this.$emit('throw-page', value)
      },
      getIndex (value) {
        let indexes = [value]
        if (value.indexOf('.') > 0) {
          indexes = value.split('.')
        }
        return indexes
      },
      getValue (object, indexes) {
        let value = null
        for (let i = 0; i < indexes.length; i++) {
          if (i !== indexes.length - 1) {
            object = object[indexes[i]]
          } else {
            if (object) {
              value = object[indexes[i]]
            }
          }
        }
        return value
      },
      sortColumn (columnName) {
        let sortA = []
        this.sort = []
        if (typeof sortA[columnName] !== 'undefined') {
          if (sortA[columnName] === 'asc') {
            sortA[columnName] = 'desc'
          } else {
            sortA[columnName] = 'asc'
          }
        } else {
          sortA[columnName] = 'asc'
        }
        this.sort = sortA
        this.$emit('sort', this.sort)
      },
      setDefaultFilterValues () {
        for(let index in this.columns) {
          let column = this.columns[index]
          if(column.filterableSettings && column.filterableSettings.defaultValue) {
            this.filter[column.name] = column.filterableSettings.defaultValue
            this.$emit('filter', this.filter)
          }
        }
      }
    },
    filters: {
      lowerCase (value) {
        if (value) {
          return value.toLowerCase()
        }
      }
    }
  }
</script>