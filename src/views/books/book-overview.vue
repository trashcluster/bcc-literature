<template>
    <section>
    <app-header :showBackButton="false" :pageName="$t('books.books')" />
    <section v-if="showIntro" id="literature-intro" class="container intro">
        <a class="minimize-button button-circular small secondary" @click="showIntro=false"></a>
        <section class="center x-small" v-html="$t('books.welcome-message')"></section>
    </section>
    <section class="container">
        <section class="center x-small">
            <search-box />
        </section>
    </section>
    <loader>
        <grid :list="publications" :title="$t('subscription.subscriptions')" :layout="GridLayout.TILES" :type="GridType.PUBLICATION"/>
        <SearchableGrids>
            <grid :list="books" :title="$t('books.books-all')" :layout="GridLayout.TILES" :type="GridType.BOOK" />
        </SearchableGrids>
    </loader>
  </section>
</template>

<script>
import { EventBus, Events } from '@/utils/eventBus';
import { mapActions, mapState } from 'vuex';
import SearchableGrids from 'components/grid/searchable-grids';
import Grid from 'components/grid/grid';
import GridMixin from '@/mixins/grid.js';
import { BookType } from '@/model/bookType.js';
import AppHeader from 'components/layout/app-header';
import SearchBox from 'components/search/search-box';
import Loader from 'components/la-loader';

export default {
    components: {
        Grid,
        SearchBox,
        Loader,
        AppHeader,
        SearchableGrids
    },
    data: function() {
        return {
            showIntro: true,
        };
    },
    mixins: [GridMixin],
    computed: {
        ...mapState('books', {
            all: state => state.base.all,
        }),
        books() {
            return this.all.filter((el) => el.bookType && el.bookType != BookType.BLAD);
        },
        publications() {
            return this.all.filter((el) => this.books.indexOf(el) < 0);
        },
    },
    methods: {
        ...mapActions('books', {
            loadAll: 'loadAll'
        }),
        async initialize() {
            await this.loadAll(false)
        },
    },
};
</script>

