<template>

  <div>

    <page-header :title="title">
      <div slot="icon">
        <mat-svg v-if="isRoot" category="action" name="explore"/>
        <mat-svg v-else category="file" name="folder"/>
      </div>
    </page-header>

    <p class="page-description" v-if="topic.description">
      {{ topic.description }}
    </p>

    <span class="visuallyhidden" v-if="subtopics.length">{{ $tr('navigate') }}</span>

    <card-list v-if="subtopics.length">
      <topic-list-item
        v-for="topic in subtopics"
        :title="topic.title"
        :link="genTopicLink(topic.id)"/>
    </card-list>

    <card-grid v-if="contents.length">
      <content-grid-item
        v-for="content in contents"
        class="card"
        :title="content.title"
        :thumbnail="content.thumbnail"
        :kind="content.kind"
        :progress="content.progress"
        :link="genContentLink(content.id)"/>
    </card-grid>

  </div>

</template>


<script>

  const getCurrentChannelObject = require('kolibri.coreVue.vuex.getters').getCurrentChannelObject;
  const PageNames = require('../../constants').PageNames;

  module.exports = {
    $trNameSpace: 'learnExplore',
    $trs: {
      explore: 'Topics',
      navigate: 'Navigate content using headings',
    },
    components: {
      'page-header': require('../page-header'),
      'topic-list-item': require('../topic-list-item'),
      'content-grid-item': require('../content-grid-item'),
      'card-grid': require('../card-grid'),
      'card-list': require('../card-list'),
    },
    computed: {
      title() {
        return this.isRoot ? this.$tr('explore') : this.topic.title;
      },
    },
    methods: {
      genTopicLink(id) {
        return {
          name: PageNames.EXPLORE_TOPIC,
          params: { channel_id: this.channelId, id },
        };
      },
      genContentLink(id) {
        return {
          name: PageNames.EXPLORE_CONTENT,
          params: { channel_id: this.channelId, id },
        };
      },
    },
    vuex: {
      getters: {
        topic: state => state.pageState.topic,
        subtopics: state => state.pageState.subtopics,
        contents: state => state.pageState.contents,
        isRoot: (state) => state.pageState.topic.id === getCurrentChannelObject(state).root_id,
        channelId: (state) => getCurrentChannelObject(state).id,
      },
    },
  };

</script>


<style lang="stylus" scoped>

  .page-description
    margin-top: 1em
    margin-bottom: 1em
    line-height: 1.5em

</style>