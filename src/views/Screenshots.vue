<template lang="pug">
div.container-fluid.py-3
  h3.mb-3 {{ $t('nav.screenshots') }}
  div(v-if="loading") {{ $t('nav.loading') }}
  div(v-else-if="error")
    b-alert(show variant="warning") {{ error }}
  div(v-else-if="shots.length === 0")
    b-alert(show variant="info") {{ $t('screenshots.empty') }}
  div(v-else)
    div.mb-2.text-muted {{ shots.length }} {{ $t('screenshots.count') }}
    div.ss-grid
      div.ss-item(v-for="s in shots", :key="s.id")
        a(:href="imgUrl(s.data.path)", target="_blank")
          img.ss-img(:src="imgUrl(s.data.path)", loading="lazy")
        div.ss-meta
          span {{ formatTime(s.timestamp) }}
          span.text-muted(v-if="s.data.monitor")  · mon {{ s.data.monitor }}
</template>

<script>
import { getClient, getStoredApiToken } from '~/util/awclient';

export default {
  name: 'Screenshots',
  data() {
    return { loading: true, error: '', shots: [] };
  },
  methods: {
    imgUrl(path) {
      const token = getStoredApiToken();
      const base = `/api/0/screenshots/${path}`;
      return token ? `${base}?token=${encodeURIComponent(token)}` : base;
    },
    formatTime(ts) {
      try {
        return new Date(ts).toLocaleString();
      } catch (e) {
        return ts;
      }
    },
  },
  mounted: async function () {
    try {
      const client = getClient();
      const buckets = await client.getBuckets();
      const ids = Object.keys(buckets).filter(
        id =>
          id.startsWith('aw-watcher-screenshot') ||
          (buckets[id] && buckets[id].type === 'cepem.screenshot')
      );
      if (ids.length === 0) {
        this.shots = [];
        return;
      }
      let all = [];
      for (const id of ids) {
        const events = await client.getEvents(id, { limit: 300 });
        all = all.concat(events.filter(e => e.data && e.data.path));
      }
      all.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
      this.shots = all;
    } catch (e) {
      this.error = 'No se pudieron cargar las capturas';
    } finally {
      this.loading = false;
    }
  },
};
</script>

<style scoped lang="scss">
.ss-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 12px;
}
.ss-item {
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  overflow: hidden;
  background: #fff;
}
.ss-img {
  width: 100%;
  height: 140px;
  object-fit: cover;
  display: block;
}
.ss-meta {
  padding: 6px 8px;
  font-size: 0.8rem;
}
</style>
