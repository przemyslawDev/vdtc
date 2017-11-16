<template>
  <div class="table-responsive">
    <label>
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
        <th v-for="column in columns">
          <div>
            {{ column.label }}
            <template v-if="column.sortable">
              <a @click="sortColumn(column.name)">
                                <span v-if="sort[column.name] === 'asc'" class="glyphicon glyphicon-sort-by-attributes"
                                      aria-hidden="true"></span>
                <span v-else-if="sort[column.name] === 'desc'"
                      class="glyphicon glyphicon-sort-by-attributes-alt" aria-hidden="true"></span>
                <span v-else class="glyphicon glyphicon-sort" aria-hidden="true"></span>
              </a>
            </template>
          </div>
          <div v-if="column.filterable">
            <template v-if="column.fliterableSettings">
              <select v-if="column.fliterableSettings.type === 'select'"
                      @change="filterSelectChange($event.target.value, column.name)">
                <option v-for="option in column.fliterableSettings.options" :value="option.value">{{
                  option.label }}
                </option>
              </select>
            </template>
            <input v-else @keyup="filterInputKeyup($event.target.value, column.name)" class="filter-input"
                   style="width:  100%;" placeholder="Search">
          </div>
          <div v-else style="height: 26px;">

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
    <pagination v-if="pagination" :info="pagination" @throwpage="throwPage"></pagination>
  </div>
</template>

<script>
  import Pagination from './Pagination.vue'

  export default {
    data () {
      return {
        sort: {},
        filter: {},
        per_page: 0
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
    mounted () {
      this.per_page = this.pagination.per_page
    },
    components: {
      'pagination': Pagination
    },
    methods: {
      perPageChange (value) {
        this.per_page = value
        this.$emit('perpage', this.per_page)
      },
      filterInputKeyup: _.debounce(function (value, columnName) {
        this.filter[columnName] = value
        this.$emit('filter', this.filter)
      }, 500),
      filterSelectChange (value, columnName) {
        this.filter[columnName] = value
        this.$emit('filter', this.filter)
      },
      throwPage (value) {
        this.$emit('throwpage', value)
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
        let sort_a = this.sort
        this.sort = []
        if (typeof sort_a[columnName] !== 'undefined') {
          if (sort_a[columnName] === 'asc') {
            sort_a[columnName] = 'desc'
          } else {
            sort_a[columnName] = 'asc'
          }
        } else {
          sort_a[columnName] = 'asc'
        }
        this.sort = sort_a
        this.$emit('sort', this.sort)
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