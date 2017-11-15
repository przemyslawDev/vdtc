<template>
  <div class="table-responsive">
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
      filterInputKeyup: _.debounce(function (value, columnName) {
        this.filter[columnName] = value
        this.$emit('filter', this.filter)
      }, 500),
      filterSelectChange (value, columnName) {
        this.filter[columnName] = value
        this.$emit('filter', this.filter)
      },
      throwPage (val) {
        this.$emit('throwpage', val)
      },
      getIndex (val) {
        let indexes = [val]
        if (val.indexOf('.') > 0) {
          indexes = val.split('.')
        }
        return indexes
      },
      getValue (object, indexes) {
        let val = null
        for (let i = 0; i < indexes.length; i++) {
          if (i !== indexes.length - 1) {
            object = object[indexes[i]]
          } else {
            if (object) {
              val = object[indexes[i]]
            }
          }
        }
        return val
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
      lowerCase (val) {
        if (val) {
          return val.toLowerCase()
        }
      }
    }
  }
</script>