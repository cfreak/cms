<template>

    <div class="w-full flex">

        <div class="flex-1"></div>

        <ul v-if="hasMultiplePages" class="flex justify-center items-center list-reset">
            <li v-if="hasPrevious" class="mx-1">
                <a @click.prevent="selectPreviousPage"><span>&laquo;</span></a>
            </li>

            <li
                v-for="(page, i) in pages"
                :key="i"
                class="mx-1"
                :class="{ 'font-bold': page == currentPage }"
            >
                <span v-if="page === 'separator'">...</span>
                <a v-else @click.prevent="selectPage(page)">{{ page }}</a>
            </li>

            <li v-if="hasNext" class="mx-1">
                <a @click.prevent="selectNextPage"><span>&raquo;</span></a>
            </li>
        </ul>

        <div class="flex flex-1">
            <div class="flex-1"></div>

            <select-input
                v-if="perPage && isPerPageEvenUseful"
                class="ml-3"
                name="perPage"
                :placeholder="__('Per Page')"
                :options="perPageOptions"
                :value="perPage"
                @input="$events.$emit('per-page-changed', $event)" />
        </div>

    </div>

</template>

<script>
import HasInputOptions from '../fieldtypes/HasInputOptions.js'

const onEachSide = 3;

export default {

    mixins: [HasInputOptions],

    props: {
        resourceMeta: {
            type: Object,
            required: true
        },
        perPage: {
            type: Number
        }
    },

    data() {
        return {
            onEachSide,
            window: onEachSide * 2
        }
    },

    computed: {

        totalPages() {
            return this.resourceMeta.last_page;
        },

        pages() {
            const els = this.elements;

            let pages = [
                els.first,
                els.slider ? 'separator' : null,
                els.slider,
                els.last ? 'separator': null,
                els.last
            ].filter(i => i !== null);

            return _.flatten(pages);
        },

        elements() {
            if (this.lastPage < (this.onEachSide * 2) + 6) {
                return this.getSmallSlider();
            }

            if (this.currentPage <= this.window) {
                return this.getSliderTooCloseToBeginning();
            } else if (this.currentPage > (this.lastPage - this.window)) {
                return this.getSliderTooCloseToEnding();
            }

            return this.getFullSlider();
        },

        currentPage() {
            return this.resourceMeta.current_page;
        },

        lastPage() {
            return this.resourceMeta.last_page;
        },

        hasMultiplePages() {
            return this.totalPages > 1;
        },

        hasPrevious() {
            return this.currentPage > 1;
        },

        hasNext() {
            return this.currentPage < this.totalPages;
        },

        perPageOptions() {
            let defaultPaginationSize = Statamic.$config.get('paginationSize');
            let defaultOptions = [10, 25, 50, 100].filter(size => size !== defaultPaginationSize);
            let options = this.normalizeInputOptions(defaultOptions);

            options.push({
                value: defaultPaginationSize,
                label: `${defaultPaginationSize} (default)`
            });

            return _.sortBy(options, 'value');
        },

        isPerPageEvenUseful() {
            return this.resourceMeta.total > this.perPageOptions[0].value;
        },

    },

    methods: {

        selectPage(page) {
            if (page === this.currentPage) {
                return;
            }

            this.$emit('page-selected', page);

            window.scrollTo(0, 0);
        },

        selectPreviousPage() {
            this.selectPage(this.currentPage - 1);
        },

        selectNextPage() {
            this.selectPage(this.currentPage + 1);
        },

        getSmallSlider() {
            return {
                first: this.getRange(1, this.lastPage),
                slider: null,
                last: null,
            }
        },

        getFullSlider() {
            return {
                first: this.getStart(),
                slider: this.getAdjacentRange(),
                last: this.getFinish(),
            }
        },

        getSliderTooCloseToBeginning() {
            return {
                first: this.getRange(1, this.window + 2),
                slider: null,
                last: this.getFinish(),
            }
        },

        getSliderTooCloseToEnding() {
            const last = this.getRange(this.lastPage - (this.window + 2), this.lastPage);

            return {
                first: this.getStart(),
                slider: null,
                last,
            }
        },

        getStart() {
            return this.getRange(1, 2);
        },

        getFinish() {
            return this.getRange(this.lastPage - 1, this.lastPage);
        },

        getAdjacentRange() {
            return this.getRange(
                this.currentPage - this.onEachSide,
                this.currentPage + this.onEachSide
            );
        },

        getRange(start, end) {
            return _.range(start, end+1);
        }

    }

}
</script>
