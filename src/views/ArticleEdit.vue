<template>
  <div class="editor-page">
    <div class="container page">
      <div class="row">
        <div class="col-md-10 offset-md-1 col-xs-12">
          <RwvListErrors :errors="errors" />
          <form v-on:submit.prevent="onPublish(project.slug);">
            <fieldset :disabled="inProgress">
              <fieldset class="form-group">
                <input
                  type="text"
                  class="form-control form-control-lg"
                  v-model="project.title"
                  placeholder="Article Title"
                />
              </fieldset>
              <fieldset class="form-group">
                <input
                  type="text"
                  class="form-control"
                  v-model="project.description"
                  placeholder="What's this project about?"
                />
              </fieldset>
              <fieldset class="form-group">
                <textarea
                  class="form-control"
                  rows="8"
                  v-model="project.body"
                  placeholder="Write your project (in markdown)"
                >
                </textarea>
              </fieldset>
              <fieldset class="form-group">
                <input
                  type="text"
                  class="form-control"
                  placeholder="Enter tags"
                  v-model="tagInput"
                  v-on:keypress.enter.prevent="addTag(tagInput);"
                />
                <div class="tag-list">
                  <span
                    class="tag-default tag-pill"
                    v-for="(tag, index) of project.tagList"
                    :key="tag + index"
                  >
                    <i class="ion-close-round" v-on:click="removeTag(tag);">
                    </i>
                    {{ tag }}
                  </span>
                </div>
              </fieldset>
            </fieldset>
            <button
              :disabled="inProgress"
              class="btn btn-lg pull-xs-right btn-primary"
              type="submit"
            >
              Publish Article
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import store from "@/store";
import RwvListErrors from "@/components/ListErrors";
import {
  ARTICLE_PUBLISH,
  ARTICLE_EDIT,
  FETCH_PROJECT,
  ARTICLE_EDIT_ADD_TAG,
  ARTICLE_EDIT_REMOVE_TAG,
  ARTICLE_RESET_STATE
} from "@/store/actions.type";

export default {
  name: "RwvArticleEdit",
  components: { RwvListErrors },
  props: {
    previousArticle: {
      type: Object,
      required: false
    }
  },
  async beforeRouteUpdate(to, from, next) {
    // Reset state if user goes from /editor/:id to /editor
    // The component is not recreated so we use to hook to reset the state.
    await store.dispatch(ARTICLE_RESET_STATE);
    return next();
  },
  async beforeRouteEnter(to, from, next) {
    // SO: https://github.com/vuejs/vue-router/issues/1034
    // If we arrive directly to this url, we need to fetch the project
    await store.dispatch(ARTICLE_RESET_STATE);
    if (to.params.slug !== undefined) {
      await store.dispatch(
        FETCH_PROJECT,
        to.params.slug,
        to.params.previousArticle
      );
    }
    return next();
  },
  async beforeRouteLeave(to, from, next) {
    await store.dispatch(ARTICLE_RESET_STATE);
    next();
  },
  data() {
    return {
      tagInput: null,
      inProgress: false,
      errors: {}
    };
  },
  computed: {
    ...mapGetters(["project"])
  },
  methods: {
    onPublish(slug) {
      let action = slug ? ARTICLE_EDIT : ARTICLE_PUBLISH;
      this.inProgress = true;
      this.$store
        .dispatch(action)
        .then(({ data }) => {
          this.inProgress = false;
          this.$router.push({
            name: "project",
            params: { slug: data.project.slug }
          });
        })
        .catch(({ response }) => {
          this.inProgress = false;
          this.errors = response.data.errors;
        });
    },
    removeTag(tag) {
      this.$store.dispatch(ARTICLE_EDIT_REMOVE_TAG, tag);
    },
    addTag(tag) {
      this.$store.dispatch(ARTICLE_EDIT_ADD_TAG, tag);
      this.tagInput = null;
    }
  }
};
</script>
