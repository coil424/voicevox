<template>
  <div
    class="accent-slider-cell"
    :style="{
      'grid-column': `1 / span ${accentPhrase.moras.length * 2 - 1}`,
    }"
  >
    <!-- div for input width -->
    <div>
      <div>
        <q-slider
          v-if="accentPhrase.moras.length > 1"
          snap
          dense
          :min="1"
          :max="accentPhrase.moras.length"
          :step="1"
          :disable="uiLocked"
          :model-value="previewAccent.currentValue.value"
          @change="changeAccent(parseInt($event))"
          @update:model-value="previewAccent.setPreviewValue(parseInt($event))"
          @wheel="
            changeAccentByScroll(
              accentPhrase.moras.length,
              accentPhrase.accent,
              $event.deltaY
            )
          "
          @pan="setPanning($event)"
        />
      </div>
    </div>
  </div>
  <div
    class="accent-draw-cell"
    :style="{
      'grid-column': `1 / span ${accentPhrase.moras.length * 2 - 1}`,
    }"
  >
    <svg :viewBox="`0 0 ${accentPhrase.moras.length * 40 - 10} 50`">
      <polyline :points="accentLine" stroke="black" fill="none" />
    </svg>
  </div>
  <template v-for="(mora, moraIndex) in accentPhrase.moras" :key="moraIndex">
    <div
      @click="uiLocked || changeAccent(moraIndex + 1)"
      :class="[
        'accent-select-cell',
        {
          'accent-select-cell-selected':
            previewAccent.currentValue.value == moraIndex + 1,
        },
      ]"
      :style="{ 'grid-column': `${moraIndex * 2 + 1} / span 1` }"
    >
      <svg width="29" height="50" viewBox="0 0 29 50">
        <line x1="14" y1="0" x2="14" y2="50" stroke-width="1" />
      </svg>
    </div>
  </template>
</template>

<script lang="ts">
import { PreviewableValue } from "@/helpers/previewableValue";
import { defineComponent, computed, ref, onMounted, onUnmounted } from "vue";

export default defineComponent({
  name: "AudioAccent",

  props: {
    accentPhrase: { type: Object, required: true },
    accentPhraseIndex: { type: Number, required: true },
    uiLocked: { type: Boolean, required: true },
  },

  emits: ["changeAccent"],

  setup(props, { emit }) {
    const previewAccent = new PreviewableValue(() => props.accentPhrase.accent);

    // detect shift key and set flag, preventing changes in intonation while scrolling around
    let shiftKeyFlag = false;

    function handleKeyPress(event: KeyboardEvent) {
      if (event.key === "Shift") shiftKeyFlag = false;
    }

    function setShiftKeyFlag(event: KeyboardEvent) {
      if (event.shiftKey) shiftKeyFlag = true;
    }

    onMounted(() => {
      window.addEventListener("keyup", handleKeyPress);
      window.addEventListener("keydown", setShiftKeyFlag);
    });

    onUnmounted(() => {
      window.removeEventListener("keyup", handleKeyPress);
      window.removeEventListener("keydown", setShiftKeyFlag);
    });

    const changeAccent = (accent: number) => {
      emit("changeAccent", props.accentPhraseIndex, accent);
    };

    const changeAccentByScroll = (
      length: number,
      accent: number,
      deltaY: number
    ) => {
      let currentAccent = accent - (deltaY > 0 ? 1 : -1);
      if (
        !props.uiLocked &&
        !shiftKeyFlag &&
        length >= currentAccent &&
        currentAccent >= 1
      )
        changeAccent(currentAccent);
    };

    const setPanning = (panningPhrase: string) => {
      if (panningPhrase === "start") {
        previewAccent.startPreview();
      } else {
        previewAccent.stopPreview();
      }
    };

    const accentLine = computed(() => {
      const accent = previewAccent.currentValue.value ?? 0;
      return [...Array(props.accentPhrase.moras.length).keys()].map(
        (index) =>
          `${index * 40 + 15} ${
            index + 1 == accent || (index != 0 && index < accent) ? 5 : 45
          }`
      );
    });

    return {
      previewAccent,
      changeAccent,
      changeAccentByScroll,
      accentLine,
      setPanning,
    };
  },
});
</script>

<style scoped lang="scss">
@use '@/styles' as global;

div {
  padding: 0px;
  &.accent-slider-cell {
    grid-row-start: 1;
    align-self: flex-end;

    margin-left: 5px;
    margin-right: 10px;
    position: relative;
    > div {
      position: absolute;
      left: 0;
      right: 0;
      bottom: 0;
      > div {
        padding-left: 10px;
        padding-right: 5px;
      }
    }
  }
  &.accent-draw-cell {
    grid-row-start: 2;
    svg line {
      stroke: black;
    }
  }
  &.accent-select-cell {
    grid-row-start: 2;
    text-align: center;
    cursor: pointer;
    svg line {
      stroke: global.$primary;
      stroke-dasharray: 3;
    }
  }
  &.accent-select-cell-selected {
    svg line {
      stroke-dasharray: none;
      stroke-width: 3;
    }
  }
}
</style>
