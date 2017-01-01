<template>
  <div>
    <div class="left">
      <Monaco
          height="200"
          theme="vs-dark"
          :code="sourceCode"
          @mounted="sourceMounted"
          @codeChange="updateSourceCode"
          >
      </Monaco>
    </div>
    <div class="right">
      <Monaco
          height="200"
          theme="vs-dark"
          :code="testCode"
          @mounted="testsMounted"
          @codeChange="updateTestCode"
          >
      </Monaco>
    </div>
    <button @click="runCodeHandler">Run Code</button>
    <button @click="testCodeHandler">Test Code</button>
    <div class="output">
      <p v-for="output in outputs">
        <span v-for="arg in output">{{arg}}</span>
      </p>
    </div>
    <div class="test-output">
      <ul>
        <li v-for="output in testOutputs" :class="output.class">{{output.msg}}</li>
      </ul>
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
      outputs: [],
      testOutputs: [],
      sourceCode: localStorage.getItem('source') || '//',
      testCode: localStorage.getItem('test') || '//'
    }
  },
  methods: {
    sourceMounted(editor) {
      this.sourceEditor = editor;
    },
    testsMounted(editor) {
      this.testEditor = editor;
    },
    updateSourceCode() {
      localStorage.setItem('source', this.sourceEditor.getValue())
    },
    updateTestCode() {
      localStorage.setItem('test', this.testEditor.getValue())
    },
    runCode(code) {
      const outputs = [];
      const windowLog = console.log;
      console.log = function() {
        const list = [...arguments];
        outputs.push(list);
        windowLog.apply(console, list);
      };
      (new Function(code))();
      return outputs;
    },
    runCodeHandler() {
      const code = this.sourceEditor.getValue();
      this.outputs = this.runCode(code);
    },
    testCodeHandler() {
      const testCode = this.testEditor.getValue();
      const sourceCode = this.sourceEditor.getValue();
      const codeArr = testCode.split('expect');
      this.testOutputs = [];
      codeArr.shift();
      const wrappedCodeArr = codeArr.map((expect, i) => {
        return {
          test: `expect${expect}`,
          wrapped: `
            ${sourceCode}
            let temp;
            try {
              temp = expect${expect};
            } catch(e) {
              temp = e;
            }
            return temp;
          `
        }
      });
      wrappedCodeArr.forEach(code => {
        const ret = (new Function(code.wrapped))();
        if (ret.constructor.name === 'AssertionError') {
          this.testOutputs.push({
            msg: ret.message,
            class: 'fail'
          });
        } else {
          this.testOutputs.push({
            msg: code.test,
            class: 'pass'
          });
        }
      });
    }
  }
};
</script>

<style>
.left, .right {
  width: 50%;
  float: left;
  box-sizing: border-box;
}
.pass {
  color: green;
}
.fail {
  color: red;
}
</style>
