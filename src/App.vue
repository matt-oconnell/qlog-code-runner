<template>
  <div>
    <Monaco
        height="600"
        theme="vs-dark"
        @mounted="onMounted"
        >
    </Monaco>
    <button @click="clickHandler">Run Code</button>
    <button @click="testCode">Test Code</button>
    <div class="output">
      <p v-for="output in outputs">
        <span v-for="arg in output">{{arg}}</span>
      </p>
    </div>
    <div id="mocha"></div>
  </div>
</template>

<script>
const Monaco = require('vue-monaco-editor');
const { expect } = require('chai');
window.expect = expect;

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
    },
    testCode() {
      const foo = 'thing';
      let b = null;
      try {
        b = expect(foo).to.equal('bar');
      } catch(e) {
        b = e
      }
      const a = expect(foo).not.to.equal('bar');
      this.outputs = [a, b];
    }
  }
};
</script>
