<template>
  <div class="categoryview-container">
    <SubCategorySwitcher :list="subCategoryList" v-if="subCategoryList.length > 1" />

    <ThreadList
      :thread-list="threadList"
      :is-loading="isLoading"
      @load-more="handleLoadMore"
    />
  </div>
</template>

<script>
import { mapState, mapActions, mapMutations } from 'vuex'

import helper from '@/helper'
import ThreadList from '../components/ThreadList/ThreadList'
import SubCategorySwitcher from '../components/SubCategorySwitcher'

export default {
  props: ['catId'],
  components: {
    ThreadList,
    SubCategorySwitcher,
  },
  metaInfo() {
    return {
      title: this.officeMode ? 'Google' : this.category.name || 'LIHKG',
    }
  },
  data() {
    return {
      canLoadMore: false,
    }
  },
  computed: {
    ...mapState({
      officeMode: state => state.app.officeMode,
      category: state => state.category.category,
      categoryList: state => state.category.categoryList,
      threadList: state => state.threadList.threadList,
      threadListType: state => state.threadList.threadListType,
      page: state => state.threadList.page,
      isLoading: state => state.threadList.isLoading,
      activeThreadId: state => state.thread.thread.thread_id,
    }),
    subCategoryList() {
      const category = this.categoryList.find(cat => +cat.cat_id === +this.catId)
      return category ? category.sub_category : []
    }
  },
  methods: {
    ...mapActions(['fetchThreadList']),
    ...mapMutations({
      setThreadList: 'SET_THREAD_LIST',
      setThreadListType: 'SET_THREAD_LIST_TYPE',
    }),
    fetchAndScrollTop() {
      this.setThreadList([])
      window.scrollTo(0, 0)
      this.fetchThreadList({
        catId: this.catId,
        ...this.$route.query,
      })
    },
    handleLoadMore() {
      if (this.canLoadMore) {
        this.fetchNextPage()
      }
    },
    fetchNextPage() {
      this.fetchThreadList({
        catId: this.catId,
        page: this.page + 1,
        ...this.$route.query,
      })
    },
  },
  watch: {
    catId: 'fetchAndScrollTop',
    '$route.query': 'fetchAndScrollTop',
    threadListType(newVal, oldVal) {
      if (oldVal !== newVal && newVal === 'category') {
        this.fetchAndScrollTop()
      }
    },
  },
  mounted() {
    this.setThreadListType('category')

    setTimeout(() => {
      this.canLoadMore = true
    }, 200)

    if (this.activeThreadId) {
      const threadItem = document.getElementById(this.activeThreadId)
      if (threadItem) {
        threadItem.classList.add('highlight')
        setTimeout(() => {
          threadItem.classList.remove('highlight')
        }, 2000)
      }
    }
  },
}
</script>
