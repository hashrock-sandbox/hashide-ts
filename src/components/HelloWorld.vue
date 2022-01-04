<script setup lang="ts">
import { ref } from 'vue'
import TabTextArea from './TabTextArea.vue'

const files = ref<FileSystemFileHandle[]>([])
const result = ref<string>('')
const resultOld = ref<string>('')
const selectedFile = ref<FileSystemFileHandle>()
const parent = ref<FileSystemDirectoryHandle>()
const parents = ref<FileSystemDirectoryHandle[]>([])
const enableParent = ref<boolean>(false)

const save = async (handle: FileSystemFileHandle | undefined) => {
  if (!handle) {
    return;
  }
  const writable = await handle.createWritable({
    keepExistingData: false,
  });
  await writable.write(result.value);
  await writable.close();
  resultOld.value = result.value;
}

const moveParent = async () => {
  parents.value.splice(parents.value.length - 1, 1);
  parent.value = parents.value[parents.value.length - 1];
  const _files = await parents.value[parents.value.length - 1].values();
  files.value = [];
  for await (const entry of _files) {
    files.value.push(entry as FileSystemFileHandle);
  }
  enableParent.value = parents.value.length > 1;
}

const open = async (f: FileSystemFileHandle | FileSystemDirectoryHandle) => {
  if (f.kind === "file") {
    var reader = new FileReader();
    reader.onload = () => {
      const _result = reader.result;
      if (_result != null) {
        result.value = _result.toString()
        resultOld.value = result.value;
      }
    };
    reader.readAsBinaryString(await f.getFile());
    selectedFile.value = f;
    return;
  } else {
    parents.value.push(f);
    const _files = await parents.value[parents.value.length - 1].values();
    files.value = [];
    for await (const entry of _files) {
      files.value.push(entry as FileSystemFileHandle);
    }
    parent.value = f;
    enableParent.value = parents.value.length > 1;
  }

}
const openDir = async () => {
  parents.value = [];
  const _parent = await window.showDirectoryPicker() as FileSystemDirectoryHandle;
  if (!_parent) {
    return;
  }
  parents.value.push(_parent);
  parent.value = _parent;
  files.value = [];
  for await (let handle of _parent.values()) {
    files.value.push(handle as FileSystemFileHandle);
  }
}
</script>

<template>
  <div class="nav">
    <div class="nav__current">
      <svg
        id="i-folder"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 32 32"
        width="24"
        height="24"
        fill="none"
        stroke="currentcolor"
        stroke-linecap="round"
        stroke-linejoin="round"
        stroke-width="2"
      >
        <path d="M2 26 L30 26 30 7 14 7 10 4 2 4 Z M30 12 L2 12" />
      </svg>
      <div class="nav__selected">
        <span v-if="parent">{{ parent.name }}</span>
      </div>
      <button @click="openDir">Open</button>
    </div>
    <ul>
      <li v-if="enableParent" @click="moveParent()">../</li>
      <li v-for="f in files" @click="open(f)" :class="{ selected: f === selectedFile }">
        {{ f.name }}
        <span v-if="f.isDirectory">/</span>
      </li>
    </ul>
  </div>

  <div class="editor">
    <div class="editor__nav" v-if="selectedFile">
      <svg
        id="i-file"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 32 32"
        width="24"
        height="24"
        fill="none"
        stroke="currentcolor"
        stroke-linecap="round"
        stroke-linejoin="round"
        stroke-width="2"
      >
        <path d="M6 2 L6 30 26 30 26 10 18 2 Z M18 2 L18 10 26 10" />
      </svg>
      <div v-if="selectedFile" class="editor__selected">
        {{ selectedFile.name }}
        <span v-if="result !== resultOld">*</span>
      </div>
      <button @click="save(selectedFile)">Save</button>
    </div>
    <TabTextArea v-model="result" @save="save(selectedFile)" />
  </div>
</template>

<style scoped>
li {
  cursor: pointer;
  list-style: none;
  padding: 0.5em;
}
html,
body {
  height: 100%;
  margin: 0;
}
#app {
  display: flex;
  height: 100%;
}
.nav {
  width: 300px;
  background: #222;
  color: white;
  padding: 0;
  overflow-y: auto;
}

ul {
  padding: 0;
  margin: 0;
  font-size: 1em;
  font-family: monospace;
}
.editor {
  flex: 1;
  display: flex;
  flex-direction: column;
}
.editor__nav {
  display: flex;
  background: #555;
  color: #58ffcd;
  align-items: center;
  padding: 0.5em;
}

textarea {
  padding: 1em;
  flex: 1;
  color: white;
  background-color: #333;
  border: none;
  outline: none;
  tab-size: 2;
  word-break: break-all;
}
button {
  background-color: #777;
  color: white;
  border: 1px solid black;
  padding: 0.5em 0.75em;
}
button:hover {
  background-color: #888;
}

.nav {
}

.nav__current {
  display: flex;
  padding: 0.5em;
  align-items: center;
}

.nav__selected {
  flex: 1;
  padding: 0 0.5em;
}

.editor__selected {
  padding: 0 0.5em;
  flex: 1;
}

li:hover {
  background: #444;
}

li.selected {
  color: #58ffcd;
}
</style>
