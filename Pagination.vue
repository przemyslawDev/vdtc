<template>
    <div class="text-center">
        <div class="text-muted">
            <span>{{ $t('Total records') }}: {{ info.total }}</span>
            <span v-if="hasPages"><br>{{ $t('Total pages') }}: {{ info.last_page }}</span>
        </div>
        <ul v-if="info.total > 0 && hasPages" class="pagination">
            <li :class="isFirstPage ? 'disabled' : ''">
                <a @click="firstPage" :aria-label="$t('First')">
                    <span aria-hidden="true"><i class="fa fa-angle-double-left"></i></span>
                </a>
            </li>
            <li :class="isFirstPage ? 'disabled' : ''">
                <a @click="previousPage" :aria-label="$t('Previous')">
                    <span aria-hidden="true"><i class="fa fa-angle-left"></i></span>
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
                <a @click="nextPage" :aria-label="$t('Next')">
                    <span aria-hidden="true"><i class="fa fa-angle-right"></i></span>
                </a>
            </li>
            <li :class="isLastPage ? 'disabled': ''">
                <a @click="lastPage" :aria-label="$t('Last')">
                    <span aria-hidden="true"><i class="fa fa-angle-double-right"></i></span>
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
                this.$emit('throw-page', this.info.current_page + 1)
            }
        },
        previousPage () {
            if (this.info.current_page !== 1) {
                this.$emit('throw-page', this.info.current_page - 1)
            }
        },
        forwardPage () {
            this.$emit('throw-page', this.info.current_page + 1)
        },
        backwardPage () {
            this.$emit('throw-page', this.info.current_page - 1)
        },
        firstPage() {
            this.$emit('throw-page', 1)
        },
        lastPage() {
            this.$emit('throw-page', this.info.last_page)
        }
    }
    }
</script>
<style>
    .active {
        color: white;
    }
    ul.pagination {
        margin-top: 10px;
    }
</style>