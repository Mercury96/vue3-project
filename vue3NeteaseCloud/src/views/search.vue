<template>
    <div class="box-wrap">
        <search-bar 
            @toSearch="toSearch"
            @clear="onClearSearch"
            :searchVal="searchVal"
            @inputChanges="onInputChanges"
            @focused="onFocus"
        />
        <section class="m-hotlist" v-show="isSearch === false">
            <h3 class="title">热门搜索</h3>
            <ul class="list">
                <li @click.stop="chooseHot(item.searchWord)" class="item f-bd f-bd-full" v-for="(item) of hotLists" :key="item.score">
                    <a class="link" href="javascript:void(0);">{{ item.searchWord }}</a>
                </li>
            </ul>
        </section>

        <section class="m-recom" v-show="isSearch === true">
            <h3 class="title f-bd f-bd-btm f-thide">搜索“{{ searchVal }}”</h3>
            <ul class="suggest-list">
                <li v-for="(item) of suggestList" :key="item.keyword">
                    <van-icon name="search" />
                    <span class="sear-name f-bd f-bd-btm f-thide">{{ item.keyword }}</span>
                </li>
            </ul>
        </section>
    </div>
</template>

<script>
    import SearchBar from '@/components/searchBar/searchBar.vue';
    import { reactive, ref } from 'vue';
    import { getHotSearch, getSuggestSearch, getSearchByKw } from '@/api/search.js';
    import { Icon } from 'vant';
    import { loading, loaded } from '@/untils/common';
    export default {
        name: 'Search',
        components: {
            'search-bar': SearchBar,
            'van-icon': Icon
        },
        setup() {

            const hotLists = ref([]);
            const searchVal = ref('');
            const isSearch = ref(false);
            const suggestList = ref([]);
            const oldSearch = ref('');
            const searchResult = ref([]);

            function toSearch(val) { //执行搜索
                searchVal.value = val.trim();
                if(val.trim() != '') {
                    isSearch.value = null;
                    getSearchByKws();
                }
            }

            function onClearSearch() { //取消搜索
                isSearch.value = false;
            }

            async function getHotSearchs() { //热门搜索列表
                loading();
                let res = await getHotSearch();
                setTimeout(() => {
                    loaded();
                }, 500)
                // console.log(res);
                hotLists.value = res;
            }

            function chooseHot(sear) { //点击选择热门搜索
                searchVal.value = sear;
                isSearch.value = null;
                getSearchByKws();
            }

            function onInputChanges(val) { //输入框变化
                searchVal.value = val.trim();
                if(val.trim() != '') {
                    isSearch.value = true;
                    getSuggestSearches(searchVal.value);
                }else {
                    isSearch.value = false;
                }
            }

            function onFocus() { //聚焦
                // console.log(searchVal.value);
                if(searchVal.value != '') {
                    isSearch.value = true;
                    if(oldSearch.value !== searchVal.value) {
                        getSuggestSearches(searchVal.value);
                    }
                }else {
                    isSearch.value = false;
                }
            }

            async function getSuggestSearches(val) { //获取建议搜索
                loading('搜索中');
                let res = await getSuggestSearch(val);
                // console.log(res);
                loaded();
                if(res.allMatch) {
                    suggestList.value = res.allMatch;
                    oldSearch.value = searchVal.value; //搜索后记录上次值
                }
            }

            async function getSearchByKws() { //根据关键字搜索的结果
                let len = searchResult.value.length;
                let res = await getSearchByKw(searchVal.value, 20, len);
                console.log(res);
            }

            getHotSearchs();

            

            return {
                toSearch,
                onClearSearch,
                hotLists,
                chooseHot,
                searchVal,
                isSearch,
                onInputChanges,
                onFocus,
                suggestList,
            }
        }
    }
</script>

<style lang="scss" scoped>
    @mixin flex($x: center, $y: space-between) {
        display: flex;
        align-items: $x;
        justify-content: $y;
    }
    @mixin elp{
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
    .m-hotlist{
        padding: 15px 10px 0;
        .title {
            font-size: 12px;
            line-height: 12px;
            color: #666;
        }
        .list {
            margin: 10px 0 7px;
        }
        .item {
            display: inline-block;
            height: 32px;
            margin-right: 8px;
            margin-bottom: 8px;
            padding: 0 14px;
            font-size: 14px;
            line-height: 32px;
            color: #333;

            &::after {
                border-color: #d3d4da;
                border-radius: 32px;
            }
        }
    }
    .f-bd {
        position: relative;

        &::after{
            position: absolute;
            z-index: 2;
            content: "";
            top: 0;
            left: 0;
            pointer-events: none;
            box-sizing: border-box;
            width: 100%;
            height: 100%;
            transform-origin: top left;
            border: 0 solid rgba(0,0,0,.1);
        }
    }
    .f-bd-full:after {
        border-width: 1px;
    }

    a{
        color: #333;
        &:visited{
            color: #333;
        }
    }

    .m-recom {
        margin: 0 auto;
        .title {
            height: 50px;
            margin-left: 10px;
            padding-right: 10px;
            font-size: 15px;
            line-height: 50px;
            color: #507daf;
            font-weight: normal;

            &::after {
                border-color: rgba(0,0,0,.1);
            }
        }
    }
    .f-thide {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        word-break: normal;
    }
    .f-bd-btm:after {
        border-bottom-width: 1px;
    }

    .suggest-list{
        li{
            height: 45px;
            @include flex(center, flex-start);
            color: #333;
            font-size: 15px;
            padding-left: 10px;
        }
    }

    .jacky-search{
        margin-right: 7px;
    }

    .sear-name{
        margin-left: 7px;
        padding-right: 10px;
        flex: 1;
        line-height: 45px;
        @include elp;
    }
</style>