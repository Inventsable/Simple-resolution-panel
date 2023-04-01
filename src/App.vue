<template>
  <div id="app">
    <Menus refresh />
    <Panel>
      <Anno v-if="error">HostAdapter not installed</Anno>
      <!-- <Anno v-else>{{ this.res }}</Anno> -->
      <!-- <Select label="Risoluzione" v-model="activeValue" :items="resolutions" @update="updateValue" /> -->
      <Button-Group label='Risoluzione' exclusive :active="activeIndex" @update="updateIndex">
        <Button label="72ppi" :color="activeIndex == 0 ? '#59e' : 'var(--color-default)'" ref="btn0" />
        <Button label="150ppi" :color="activeIndex == 1 ? '#59e' : 'var(--color-default)'" ref="btn1" />
        <Button label="300ppi" :color="activeIndex == 2 ? '#59e' : 'var(--color-default)'" ref="btn2" />
      </Button-Group>
    </Panel>
  </div>
</template>

<script>
import { evalScript } from "brutalism";
export default {
  data: () => ({
    res: 72,
    resolutions: [
      { value: '72', label: 'Video (72 ppi)' },
      { value: '150', label: 'Media (150 ppi)' },
      { value: '300', label: 'Alta (300 ppi)' }
    ],
    activeValue: '72',
    override: false,
    error: false,
    isMounted: false,
    activeIndex: 0,
  }),
  async mounted() {
    const self = this;
    console.log(AIEventAdapter)
    self.activeValue = await evalScript(`(function() { return app.activeDocument.rasterEffectSettings.resolution }())`) + ""
    const adapter = AIEventAdapter.getInstance()
    adapter.addEventListener(
      AIEvent.ART_SELECTION_CHANGED,
      async (e) => {
        self.override = true;
        self.activeValue = await evalScript(`(function() { return app.activeDocument.rasterEffectSettings.resolution }())`) + ""
        self.$nextTick(() => {
          self.override = false;
        })
      }

    );
    adapter.addEventListener(
      AIEvent.MENU_CHANGED,
      async (e) => {
        self.override = true;
        self.activeValue = await evalScript(`(function() { return app.activeDocument.rasterEffectSettings.resolution }())`) + ""
        self.$nextTick(() => {
          self.override = false;
        })
      }
    );
    this.isMounted = true;
    this.$nextTick(() => {
      self.flushActiveElts()
    })
    // 
  },
  watch: {
    activeValue(value) {
      console.log(value);
      this.activeIndex = [72, 150, 300].findIndex(i => +i == +value);
      if (this.isMounted) {
        this.flushActiveElts()
      }
    },
  },
  methods: {
    flushActiveElts() {
      [0, 1, 2].forEach(i => {
        const ref = this.$refs[`btn${i}`].$el
        if (+i !== +this.activeIndex) {
          ref.classList.remove('active');
        } else {
          ref.classList.add('active');
        }
      })
    },
    async updateValue(value) {
      const self = this;
      if (!self.override) {
        console.log('UPDATE:', value);
        self.override = true;
        await evalScript(`runActionFromString('${value}ppi', 'resolution')`);
        setTimeout(() => {
          self.override = false;
        }, 500);
      }
    },
    async updateIndex(value) {
      const self = this;
      const reso = [
        72, 150, 300
      ]
      if (!self.override) {
        self.activeIndex = value;
        const e = reso[value];
        console.log('UPDATE:', value);
        self.override = true;
        await evalScript(`runActionFromString('${e}ppi', 'resolution')`);
        setTimeout(() => {
          self.override = false;
        }, 500);
      }
    }
  },
};
</script>

<style>
.app,
.panel,
html,
body {
  overflow-y: hidden;
}
</style>
