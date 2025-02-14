<!--
Copyright 2019 Google Inc. All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->
<template>
  <div>

    <section class="section">
      <div class="container is-fluid">
        <ts-navbar-secondary v-if="sketch.active_timelines.length" currentAppContext="sketch" currentPage="overview">
          <a class="button is-info is-rounded" style="margin-right:10px;">
              <span class="icon is-small">
                <i class="fas fa-users"></i>
              </span>
            <span>Share</span>
          </a>
          <div class="dropdown is-hoverable is-right" v-bind:class="{'is-active': settingsDropdownActive}">
            <div class="dropdown-trigger">
              <a class="button" style="background:transparent;border:none;" aria-haspopup="true" aria-controls="dropdown-menu" v-on:click="settingsDropdownActive = !settingsDropdownActive">
                <span>More</span>
                <span class="icon is-small">
                <i class="fas fa-angle-down" aria-hidden="true"></i>
              </span>
              </a>
            </div>
            <div class="dropdown-menu" id="dropdown-menu" role="menu">
              <div class="dropdown-content">
                <a v-if="meta.permissions.delete" class="dropdown-item" v-on:click="showDeleteSketchModal = !showDeleteSketchModal">
                  <span class="icon is-small" style="margin-right:5px;"><i class="fas fa-trash"></i></span>
                  <span>Delete</span>
                </a>
              </div>
            </div>
          </div>
        </ts-navbar-secondary>
      </div>
    </section>

    <b-modal :active.sync="showUploadTimelineModal" :width="640" scroll="keep">
      <div class="card">
        <header class="card-header">
          <p class="card-header-title">Upload new timeline</p>
        </header>
        <div class="card-content">
          <div class="content">
            <p>
              Supported formats are Plaso storage file, JSONL, or a CSV file.
              If you are uploading a CSV or JSONL file make sure to read the <a href="https://github.com/google/timesketch/blob/master/docs/Users-Guide.md#adding-timelines" rel="noreferrer" target="_blank">documentation</a> to learn what columns are needed.
            </p>
            <ts-upload-timeline-form @toggleModal="showUploadTimelineModal = !showUploadTimelineModal"></ts-upload-timeline-form>
          </div>
        </div>
      </div>
    </b-modal>

    <b-modal :active.sync="showDeleteSketchModal" :width="640" scroll="keep">
      <div class="card">
        <header class="card-header">
          <p class="card-header-title">Delete sketch</p>
        </header>
        <div class="card-content">
          <div class="content">
            <p>Are you sure you want to delete this sketch?</p>
            <div class="field is-grouped">
              <p class="control">
                <button class="button is-danger" v-on:click="deleteSketch">
                  <span class="icon is-small" style="margin-right:5px;"><i class="fas fa-trash"></i></span>
                  <span>Delete</span>
                </button>
              </p>
              <p class="control">
                <button class="button" v-on:click="showDeleteSketchModal = !showDeleteSketchModal">
                  <span>I changed my mind, keep the sketch!</span>
                </button>
              </p>
            </div>
          </div>
        </div>
      </div>
    </b-modal>

    <!-- Title and description -->
    <section class="section">
      <div class="container is-fluid">
        <div class="card" style="min-height: 200px;">
          <div class="card-content">
            <ts-sketch-summary :sketch="sketch"></ts-sketch-summary>
          </div>
        </div>
      </div>
    </section>

    <!-- Stats -->
    <section class="section" v-if="sketch.timelines.length">
      <div class="container is-fluid">
        <div class="card" style="min-height: 100px;">
          <div class="card-content">
            <ts-sketch-metrics :timelines="sketch.active_timelines" :views="meta.views" :stories="sketch.stories" :count="count"></ts-sketch-metrics>
          </div>
        </div>
      </div>
    </section>

    <!-- Timeline, Saved View and Stories lists-->
    <section class="section" v-if="sketch.timelines && sketch.timelines.length ? sketch.timelines.length: false">
      <div class="container is-fluid">
        <div class="columns">

          <!-- Timelines -->
          <div class="column" v-if="sketch.timelines && sketch.timelines.length ? sketch.timelines.length: false">
            <div class="card has-min-height">
              <header class="card-header">
                <p class="card-header-title">Timelines</p>
                <div class="field is-grouped is-pulled-right" style="padding: 0.75rem;">
                  <p class="control">
                    <button class="button is-success is-rounded is-small" v-on:click="showUploadTimelineModal = !showUploadTimelineModal">
                        <span class="icon is-small">
                          <i class="fas fa-plus"></i>
                        </span>
                      <span>Timeline</span>
                    </button>
                  </p>
                  <p class="control">
                    <router-link class="button is-rounded is-small" :to="{ name: 'SketchManageTimelines' }">
                      <span class="icon is-small">
                        <i class="fas fa-cog"></i>
                      </span>
                      <span>Manage</span>
                    </router-link>
                  </p>
                </div>
              </header>
              <div class="card-content" style="padding:5px;">
                <ts-timeline-list :timelines="sketch.timelines" :controls="false"></ts-timeline-list>
              </div>
            </div>
          </div>

          <!-- Saved views -->
          <div class="column" v-if="meta.views && meta.views.length ? meta.views.length: false">
            <div class="card has-min-height">
              <header class="card-header">
                <p class="card-header-title">Views</p>
                <div class="field is-grouped is-pulled-right" style="padding: 0.75rem;">
                  <p class="control">
                    <router-link class="button is-rounded is-small" :to="{ name: 'SketchManageViews' }">
                      <span class="icon is-small">
                        <i class="fas fa-cog"></i>
                      </span>
                      <span>Manage</span>
                    </router-link>
                  </p>
                </div>
              </header>
              <div class="card-content" style="padding:5px;">
                <ts-saved-view-list :views="meta.views"></ts-saved-view-list>
              </div>
            </div>
          </div>

          <!-- Stories -->
          <div class="column" v-if="sketch.stories && sketch.stories.length ? sketch.stories.length: false">
            <div class="card has-min-height">
              <header class="card-header">
                <p class="card-header-title">Stories</p>
              </header>
              <div class="card-content" style="padding:5px;">
                <ts-sketch-story-list></ts-sketch-story-list>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <ts-sketch-timelines-manage v-if="!sketch.timelines.length" :hide-navigation="true"></ts-sketch-timelines-manage>

  </div>
</template>

<script>
import ApiClient from '../utils/RestApiClient'
import TsSketchSummary from '../components/Sketch/SketchSummary'
import TsSketchMetrics from '../components/Sketch/SketchMetrics'
import TsTimelineList from '../components/Sketch/TimelineList'
import TsSavedViewList from '../components/Sketch/ViewList'
import TsSketchStoryList from '../components/Sketch/StoryList'
import TsUploadTimelineForm from '../components/Sketch/UploadForm'
import TsSketchTimelinesManage from './SketchManageTimelines'

export default {
  components: {
    TsSketchMetrics,
    TsSketchSummary,
    TsTimelineList,
    TsSavedViewList,
    TsUploadTimelineForm,
    TsSketchStoryList,
    TsSketchTimelinesManage
  },
  data () {
    return {
      settingsDropdownActive: false,
      showUploadTimelineModal: false,
      showDeleteSketchModal: false
    }
  },
  computed: {
    sketch () {
      return this.$store.state.sketch
    },
    meta () {
      return this.$store.state.meta
    },
    count () {
      return this.$store.state.count
    }
  },
  methods: {
    deleteSketch: function () {
      ApiClient.deleteSketch(this.sketch.id).then((response) => {
        this.$router.push({ name: 'Home' })
      }).catch((e) => {
        console.error(e)
      })
    }
  }

}
</script>

<style lang="scss">
  .has-min-height {
    min-height: 300px;
  }
</style>
