<template>
    <div>
      <h1>
        <transition name="fade" mode="out-in">
          <span :key="title">
            {{ title }}
          </span>
        </transition>
      </h1>
      <transition name="fade" mode="out-in">
        <component :is="article" />
      </transition>
    </div>

</template>

<script>
export default {
  data() {
    return {
      article: null,
    };
  },
  computed: {
    route() {
      return this.$route.params.page;
    },
    title() {
      return this.route.split('-').join(' ');
    },
  },
  created() {
    if (this.route) {
      this.getArticle();
    }
  },
  methods: {
    getArticle() {
      this.article = require(`@/wiki/${this.route}.md`).default;

      this.$nextTick().then(() => {
        document.querySelectorAll('.wiki a').forEach(link => {
          link.addEventListener('click', event => {
            event.preventDefault();

            let target;

            if (['STRONG', 'CODE'].includes(event.target.tagName)) {
              target = event.target.parentNode;
            } else if (event.target.tagName === 'A') {
              target = event.target;
            }

            if (target.href.startsWith(window.origin)) {
              this.$router.push({
                path: target.pathname,
                hash: target.hash
              }).then().catch(() => {});
            } else {
              window.open(target.href);
            }
          });
        });

        if (this.$route.hash) {
          this.scrollTo(this.$route.hash);
        }
      });
    },
    scrollTo(hash) {
      const element = document.getElementById(hash.split('#')[1]);
      element.scrollIntoView({
        behavior: 'smooth',
      });
    },
  },
  watch: {
    $route(to) {
      if (to.hash) {
        this.$nextTick().then(() => {
          this.scrollTo(to.hash);
        });
      } else {
        document.getElementById('article').scrollTo({
          top: 0,
          behavior: 'smooth'
        });
      }
      this.getArticle();
    }
  },
};
</script>
