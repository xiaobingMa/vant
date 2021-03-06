<template>
  <div :class="b()">
    <slot />
    <div v-show="loading" :class="b('loading')">
      <slot name="loading">
        <loading />
        <span :class="b('loading-text')">{{ loadingText || $t('loadingTip') }}</span>
      </slot>
    </div>
  </div>
</template>

<script>
import create from '../utils/create';
import utils from '../utils/scroll';
import { on, off } from '../utils/event';

export default create({
  name: 'list',

  model: {
    prop: 'loading'
  },

  props: {
    loading: Boolean,
    finished: Boolean,
    immediateCheck: {
      type: Boolean,
      default: true
    },
    offset: {
      type: Number,
      default: 300
    },
    loadingText: String
  },

  mounted() {
    this.scroller = utils.getScrollEventTarget(this.$el);
    this.handler(true);

    if (this.immediateCheck) {
      this.$nextTick(this.onScroll);
    }
  },

  destroyed() {
    this.handler(false);
  },

  activated() {
    /* istanbul ignore next */
    this.handler(true);
  },

  deactivated() {
    /* istanbul ignore next */
    this.handler(false);
  },

  watch: {
    loading() {
      this.$nextTick(this.onScroll);
    },

    finished() {
      this.$nextTick(this.onScroll);
    }
  },

  methods: {
    onScroll() {
      if (this.loading || this.finished) {
        return;
      }

      const el = this.$el;
      const { scroller } = this;
      const scrollerHeight = utils.getVisibleHeight(scroller);

      /* istanbul ignore next */
      if (!scrollerHeight || utils.getComputedStyle(el).display === 'none') {
        return;
      }

      const scrollTop = utils.getScrollTop(scroller);
      const targetBottom = scrollTop + scrollerHeight;

      let reachBottom = false;

      /* istanbul ignore next */
      if (el === scroller) {
        reachBottom = scroller.scrollHeight - targetBottom < this.offset;
      } else {
        const elBottom =
          utils.getElementTop(el) -
          utils.getElementTop(scroller) +
          utils.getVisibleHeight(el);
        reachBottom = elBottom - scrollerHeight < this.offset;
      }

      /* istanbul ignore else */
      if (reachBottom) {
        this.$emit('input', true);
        this.$emit('load');
      }
    },

    handler(bind) {
      /* istanbul ignore else */
      if (this.binded !== bind) {
        this.binded = bind;
        (bind ? on : off)(this.scroller, 'scroll', this.onScroll);
      }
    }
  }
});
</script>
