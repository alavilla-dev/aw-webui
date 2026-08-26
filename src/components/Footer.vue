<template lang="pug">
div.container(style="color: #555; font-size: 0.9em")
  div.mb-2
    | {{ $t('footer.madeWith') }}
    a(href="https://cepem.com.ar", target="_blank" rel="noopener noreferrer")
      icon(name="heart" scale=0.75 style="fill: #E55")
    | {{ $t('footer.byDevs') }}
    div
      span.mt-2(v-if="info", style="color: #888; font-size: 0.8em")
        span.mr-2
          b {{ $t('footer.host') }}
          | &nbsp; {{info.hostname}}
        span
          b {{ $t('footer.version') }}
          | &nbsp; {{info.version}}

  div(style="font-size: 0.9em; opacity: 0.8; fill: #88F")
    div.float-none.float-md-right.my-2
      a(href="https://github.com/alavilla-dev/cepem-watch/issues/new/choose", target="_blank" rel="noopener noreferrer")
        icon(name="bug")
        | {{ $t('footer.reportBug') }}
</template>

<script lang="ts">
import 'vue-awesome/icons/bug';
import 'vue-awesome/icons/heart';

import { mapState } from 'pinia';
import { useServerStore } from '~/stores/server';

export default {
  name: 'Footer',
  computed: {
    ...mapState(useServerStore, ['info']),
  },
};
</script>
