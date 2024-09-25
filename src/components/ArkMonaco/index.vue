<template>
  <div ref="arkEditorRef" style="height: 300px;"></div>
</template>

<script setup>
import * as monaco from 'monaco-editor';

const arkEditorRef = ref(null);
let editorInstance = null;

const props = defineProps({
  editorValue: {
    required: false,
    default: ''
  },
  editorLang: {
    required: false,
    default: 'json'
  },
  onChange: {
    required: false
  }
});

onMounted(() => {
  // 初始化 Monaco Editor
  editorInstance = monaco.editor.create(arkEditorRef.value, {
    value: props.editorValue,
    language: props.editorLang,
    theme: "vs",
    minimap: {
      size: 'fill'
    }
  });

  editorInstance.onDidChangeModelContent(() => {
    if (props.onChange) {
      props.onChange(editorInstance.getValue());
    } else {
      console.log(editorInstance.getValue());
    }
  });

  if (editorInstance) {
    editorInstance.getAction('editor.action.formatDocument').run();
  }
  console.log('ArkMonaco 初始化...【' + props.editorLang + '】 Worker');
});

onBeforeUnmount(() => {
  // 在组件销毁之前销毁编辑器
  if (editorInstance) {
    editorInstance.dispose();
    console.log('ArkMonaco ' + props.editorLang + ' 实例销毁');
  }
});
</script>

<style>
/* 确保编辑器容器有高度 */
div[ref="arkEditorRef"] {
  height: 100vh;
  border-radius: 10px;
}

</style>
