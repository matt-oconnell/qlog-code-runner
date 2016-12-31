<template>
  <div>
    <Monaco
        height="600"
        theme="vs-dark"
        @mounted="onMounted"
        >
    </Monaco>
    <button @click="clickHandler">Run Code</button>
    <div class="output">
      <p v-for="output in outputs">
        <span v-for="arg in output">{{arg}}</span>
      </p>
    </div>
  </div>
</template>

<script>
const Monaco = require('vue-monaco-editor');

module.exports = {
  components: {
    Monaco
  },
  data: function() {
    return {
      outputs: []
    }
  },
  methods: {
    onMounted(editor) {
      this.editor = editor;
    },
    clickHandler() {
      const code = this.editor.getValue();
      const runnerLog = [];
      const windowLog = console.log;
      console.log = function() {
        const list = [...arguments];
        runnerLog.push(list);
        windowLog.apply(console, list);
      };
      (new Function(code))();
      this.outputs = runnerLog;
    }
  }
};
</script>
