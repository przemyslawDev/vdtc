<template>
  <div v-if="hasPages">
    <ul v-if="info.total > 0" class="pagination">
      <li :class="isFirstPage ? 'disabled' : ''">
        <a @click="previousPage" aria-label="Previous">
          <span aria-hidden="true">&laquo;</span>
        </a>
      </li>
      <li v-if="getBackwardPage">
        <a @click="backwardPage">{{ getBackwardPage }}
          <span class="sr-only">(current)</span>
        </a>
      </li>
      <li class="active">
        <a>{{ info.current_page }}
          <span class="sr-only">(current)</span>
        </a>
      </li>
      <li v-if="getForwardPage">
        <a @click="forwardPage">{{ getForwardPage }}
          <span class="sr-only">(current)</span>
        </a>
      </li>
      <li :class="isLastPage ? 'disabled' : ''">
        <a @click="nextPage" aria-label="Next">
          <span aria-hidden="true">&raquo;</span>
        </a>
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    props: ['info'],
    computed: {
      getForwardPage () {
        let page = null
        if (this.info.current_page < this.info.last_page) {
          page = this.info.current_page + 1
        }
        return page
      },
      getBackwardPage () {
        let page = null
        if (this.info.current_page > 1) {
          page = this.info.current_page - 1
        }
        return page
      },
      isLastPage () {
        return this.info.current_page >= this.info.last_page
      },
      isFirstPage () {
        return this.info.current_page <= 1
      },
      hasPages () {
        return this.info.last_page > 1
      }
    },
    methods: {
      nextPage () {
        if (this.info.current_page !== this.info.last_page) {
          this.$emit('throwpage', this.info.current_page + 1)
        }
      },
      previousPage () {
        if (this.info.current_page !== 1) {
          this.$emit('throwpage', this.info.current_page - 1)
        }
      },
      forwardPage () {
        this.$emit('throwpage', this.info.current_page + 1)
      },
      backwardPage () {
        this.$emit('throwpage', this.info.current_page - 1)
      },
    }
  }
</script>