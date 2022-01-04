<script lang="ts">
import { defineComponent } from 'vue';
export default defineComponent({
  props: {
    modelValue: {
      type: String,
      default: "",
    },
  },
  methods: {
    onTab(e: KeyboardEvent) {
      const obj = e.target as HTMLTextAreaElement;
      if (!obj) {
        return;
      }
      var cursorPosition = obj.selectionStart;
      var cursorLeft = obj.value.substr(0, cursorPosition);
      var cursorRight = obj.value.substr(
        cursorPosition,
        obj.value.length
      );
      obj.value = cursorLeft + "\t" + cursorRight;
      obj.selectionEnd = cursorPosition + 1;
    },
    onInput(e: Event) {
      if (e.target === null) {
        return;
      }
      const obj = e.target as HTMLTextAreaElement;
      this.$emit("update:modelValue", obj.value)
    },
    onSave(e: KeyboardEvent) {
      this.$emit("save", e)
    }
  }
})

</script>
<template>
  <textarea
    :value="modelValue"
    @input="onInput($event)"
    spellcheck="false"
    @keydown.tab.prevent="onTab"
    @keydown.ctrl.s.prevent="onSave"
  ></textarea>
</template>
