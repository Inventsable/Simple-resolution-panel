<template>
  <div id="app">
    <Menus refresh />
    <Panel debug>
      <Anno v-if="error">HostAdapter not installed</Anno>
      <Anno v-else>{{ this.res }}</Anno>
    </Panel>
  </div>
</template>

<script>
import { evalScript } from "brutalism";
export default {
  data: () => ({
    res: 72,
    error: false
  }),
  async mounted() {
    const self = this;
    console.log(AIEventAdapter)
    self.res = Number(await evalScript(`(function() { return app.activeDocument.rasterEffectSettings.resolution }())`));
    AIEventAdapter.getInstance().addEventListener(
      AIEvent.ART_SELECTION_CHANGED,
      async (e) => (
        self.res = Number(await evalScript(`(function() { return app.activeDocument.rasterEffectSettings.resolution }())`))
      )
    );
  },
  methods: {
  },
};
</script>

<style></style>
