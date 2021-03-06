<template>
  <div class="container">
    <div v-if="isLoading && !threadList.length">
      <placeholder v-for="n in numberOfPlaceholder" :key="n"></placeholder>
    </div>
    <ul class="thread-list" v-else>
      <ThreadListItem
        v-for="thread in threadList"
        :key="thread.thread_id"
        :thread="thread"
        :is-visited="thread.thread_id in history"
        :last-read-page="thread.thread_id in history && history[thread.thread_id].page"
        :last-read-post-id="thread.thread_id in history && history[thread.thread_id].postId"
        :new-reply="thread.thread_id in history && thread.no_of_reply - history[thread.thread_id].replies"
      />
    </ul>

    <div v-if="hasMore" v-observe-visibility="onEndReached"></div>

    <div class="end-of-list">
      <template v-if="hasMore">
        <Placeholder v-for="n in 2" :key="n" />
      </template>
      <span v-else>完</span>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'

import ThreadListItem from './ThreadListItem'
import Placeholder from './Placeholder'

const placeholderHeight = 80

export default {
  props: ['threadList', 'isLoading'],
  components: {
    ThreadListItem,
    Placeholder,
  },
  computed: {
    ...mapState({
      history: state => state.app.history,
      hasMore: state => state.threadList.hasMore,
    }),
    numberOfPlaceholder() {
      return Math.ceil(window.screen.height / placeholderHeight)
    },
  },
  methods: {
    onEndReached(isVisible) {
      if (this.isLoading) return
      if (!isVisible) return
      this.$emit('load-more')
    },
  },
}
</script>

<style lang="scss" scoped>
.container {
  padding-bottom: 3rem;
  padding-bottom: calc(3rem + constant(safe-area-inset-bottom));
  padding-bottom: calc(3rem + env(safe-area-inset-bottom));
}

.thread-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.end-of-list {
  padding: 1rem;
  text-align: center;
}
</style>
