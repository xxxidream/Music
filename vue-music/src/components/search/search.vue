<template>
  <div class="search">
    <div class="search-box-wrapper">
      <search-box ref="searchBox" @query="onQueryChange"></search-box>
    </div>
    <div ref="shortcutWrapper" class="shortcut-wrapper" v-show="!query">
      <Scroll class="shortcut" ref="shortcut" :data="shortcut" :refreshDelay="refreshDelay">
        <div>
        <div class="hot-key">
          <h1 class="title">热门搜索</h1>
          <ul>
            <li class="item" v-for="item in hotKey"  @click="addQuery(item.k)">
              <span>{{item.k}}</span>
            </li>
          </ul>
        </div>
        <div class="search-history" v-show="searchHistory.length">
          <h1 class="title">
            <span class="text">搜索历史</span>
            <span class="clear" @click="showComfirm">
              <i class="icon-clear"></i>
            </span>
          </h1>
          <search-list :searches="searchHistory"
                       @select="addQuery"
                       @delete="deleteOne"
          ></search-list>
        </div>
        </div>
      </Scroll>
    </div>
    <div ref="searchResult" class="search-result" v-show="query">
      <suggest @listScroll="blurInput"
               :query="query"
               @select="saveSearch"
               ref="suggest"
                ></suggest>
    </div>
    <comfirm ref="comfirm" @confirm="deleteAll" text="是否清空所有搜索历史" confirmBtnText="清空"></comfirm>
    <router-view></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
  import SearchBox from 'base/search-box/search-box'
  import Comfirm from 'base/confirm/confirm'
  import Scroll from 'base/scroll/scroll'
  import SearchList from 'base/search-list/search-list'
  import Suggest from 'components/suggest/suggest'
  import {getHotKey} from 'api/search'
  import {ERR_OK} from 'api/config'
  import {mapActions} from 'vuex'
  import {playListMixin, searchMixin} from 'common/js/mixin'

  export default {
    mixins: [playListMixin, searchMixin],
    data() {
      return {
        hotKey: [],
        showFlag: false
      }
    },
    computed: {
      shortcut() {
        return this.hotKey.concat(this.searchHistory)
      }
    },
    components: {
      SearchBox,
      Suggest,
      SearchList,
      Comfirm,
      Scroll
    },
    created() {
      this._getHotKey()
    },
    methods: {
      handlePlayList(playList) {
        const bottom = playList.length > 0 ? '60px' : ''
        this.$refs.searchResult.style.bottom = `${bottom}`
        this.$refs.shortcutWrapper.style.bottom = `${bottom}`
        this.$refs.shortcut.refresh()
        this.$refs.suggest.refresh()
      },
      showComfirm() {
        this.$refs.comfirm.show()
      },
      deleteAll() {
        this.clearSearchHistory()
      },
      deleteOne(item) {
        this.deleteSearchHistory(item)
      },
      _getHotKey() {
        getHotKey().then((res) => {
          if (res.code === ERR_OK) {
            this.hotKey = res.data.hotkey.slice(0, 10)
          }
        })
      },
      ...mapActions([
        'clearSearchHistory'
      ])
    },
    watch: {
      query(newQuery) {
        if (!newQuery) {
          setTimeout(() => {
            this.$refs.shortcut.refresh()
          }, 20)
        }
      }
    }

  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .search
    .search-box-wrapper
      margin: 20px
    .shortcut-wrapper
      position: fixed
      top: 178px
      bottom: 0
      width: 100%
      .shortcut
        height: 100%
        overflow: hidden
        .hot-key
          margin: 0 20px 20px 20px
          .title
            margin-bottom: 20px
            font-size: $font-size-medium
            color: $color-text-l
          .item
            display: inline-block
            padding: 5px 10px
            margin: 0 20px 10px 0
            border-radius: 6px
            background: $color-highlight-background
            font-size: $font-size-medium
            color: $color-text-d
        .search-history
          position: relative
          margin: 0 20px
          .title
            display: flex
            align-items: center
            height: 40px
            font-size: $font-size-medium
            color: $color-text-l
            .text
              flex: 1
            .clear
              extend-click()
              .icon-clear
                font-size: $font-size-medium
                color: $color-text-d
    .search-result
      position: fixed
      width: 100%
      top: 178px
      bottom: 0
</style>
