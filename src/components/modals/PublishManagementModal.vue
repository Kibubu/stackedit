<template>
  <modal-inner class="modal__inner-1--publish-management" aria-label="Manage publication locations">
    <div class="modal__content">
      <div class="modal__image">
        <icon-upload></icon-upload>
      </div>
      <p v-if="publishLocations.length"><b>{{currentFileName}}</b> 被发布到了以下位置:</p>
      <p v-else><b>{{currentFileName}}</b> 还没有被发布.</p>
      <div>
        <div class="publish-entry flex flex--column" v-for="location in publishLocations" :key="location.id">
          <div class="publish-entry__header flex flex--row flex--align-center">
            <div class="publish-entry__icon flex flex--column flex--center">
              <icon-provider :provider-id="location.providerId"></icon-provider>
            </div>
            <div class="publish-entry__description">
              {{location.description}}
            </div>
            <div class="publish-entry__buttons flex flex--row flex--center">
              <button class="publish-entry__button button" @click="remove(location)" v-title="'删除位置'">
                <icon-delete></icon-delete>
              </button>
            </div>
          </div>
          <div class="publish-entry__row flex flex--row flex--align-center">
            <div class="publish-entry__url">
              {{location.url}}
            </div>
            <div class="publish-entry__buttons flex flex--row flex--center" v-if="location.url">
              <button class="publish-entry__button button" v-clipboard="location.url" @click="info('位置URL已复制到剪贴板!')" v-title="'复制URL'">
                <icon-content-copy></icon-content-copy>
              </button>
              <a class="publish-entry__button button" v-if="location.url" :href="location.url" target="_blank" v-title="'打开位置'">
                <icon-open-in-new></icon-open-in-new>
              </a>
            </div>
          </div>
          <div class="publish-entry__row flex flex--row flex--align-center" v-if="shareUrl(location)">
            <div class="publish-entry__url">
              分享链接: {{shareUrl(location)}}
            </div>
            <div class="publish-entry__buttons flex flex--row flex--center">
              <button class="publish-entry__button button" v-clipboard="shareUrl(location)" @click="info('分享URL已复制到剪贴板!')" v-title="'复制分享URL'">
                <icon-content-copy></icon-content-copy>
              </button>
              <a class="publish-entry__button button" :href="shareUrl(location)" target="_blank" v-title="'打开分享'">
                <icon-open-in-new></icon-open-in-new>
              </a>
            </div>
          </div>
        </div>
      </div>
      <div class="modal__info" v-if="publishLocations.length">
        <b>提示:</b> 删除位置不会删除任何文件。
      </div>
    </div>
    <div class="modal__button-bar">
      <button class="button button--resolve" @click="config.resolve()">关闭</button>
    </div>
  </modal-inner>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import ModalInner from './common/ModalInner';
import store from '../../store';
import badgeSvc from '../../services/badgeSvc';

export default {
  components: {
    ModalInner,
  },
  computed: {
    ...mapGetters('modal', [
      'config',
    ]),
    ...mapGetters('publishLocation', {
      publishLocations: 'current',
    }),
    currentFileName() {
      return store.getters['file/current'].name;
    },
  },
  methods: {
    ...mapActions('notification', [
      'info',
    ]),
    remove(location) {
      store.commit('publishLocation/deleteItem', location.id);
      badgeSvc.addBadge('removePublishLocation');
    },
    shareUrl(location) {
      if (location.providerId !== 'giteegist') {
        return null;
      }
      if (!location.url) {
        return null;
      }
      const splitIndex = location.url.lastIndexOf('/');
      return `${window.location.protocol}//${window.location.host}/share.html?id=${location.url.substr(splitIndex + 1)}`;
    },
  },
};
</script>

<style lang="scss">
@import '../../styles/variables.scss';

.publish-entry {
  margin: 1.5em 0;
  height: auto;
  font-size: 17px;
  line-height: 1.5;
}

$button-size: 30px;
$small-button-size: 22px;

.publish-entry__header {
  line-height: $button-size;
}

.publish-entry__row {
  border-top: 1px solid $hr-color;
  line-height: $small-button-size;
}

.publish-entry__icon {
  height: 22px;
  width: 22px;
  margin-right: 0.75rem;
  flex: none;
}

.publish-entry__description {
  width: 100%;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.publish-entry__url {
  width: 100%;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  opacity: 0.5;
  font-size: 0.67em;
}

.publish-entry__buttons {
  margin-left: 0.75rem;

  .publish-entry__row & {
    margin-left: 0.5rem;
  }
}

.publish-entry__button {
  width: $button-size;
  height: $button-size;
  padding: 4px;
  background-color: transparent;
  opacity: 0.75;

  .publish-entry__row & {
    width: $small-button-size;
    height: $small-button-size;
    padding: 4px;
  }

  &:active,
  &:focus,
  &:hover {
    opacity: 1;
    background-color: rgba(0, 0, 0, 0.1);
  }
}
</style>
