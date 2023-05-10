<template>
  <div class="container">
    <div class="edit-container">
      <div class="config-bar">
        <button @click="format">格式化</button>
        <div class="gap"></div>
        <button @click="jsonContent = '{}'">重置</button>
        <div class="gap"></div>
        <div class="input-control">
          <label for="rootInterfaceName">名称: </label>
          <input id="rootInterfaceName" class="item" type="text" v-model="rootInterfaceName"
            placeholder="root interface name" />
        </div>
        <button class="transform-btn" @click="getTransformedCode">转换</button>
      </div>
      <textarea ref="inputRef" class="editor" placeholder="输入或粘贴 JSON 或 JS 对象" v-model="jsonContent"></textarea>
    </div>
    <div class="preview-container">
      <div class="config-bar">
        <div class="input-control">
          <label for="indentNumber">缩进字符: </label>
          <select id="indentNumber" v-model="indentNumber">
            <option disabled value="">请选择</option>
            <option>2</option>
            <option>4</option>
          </select>
        </div>
        <div class="gap"></div>
        <div class="input-control">
          <label for="exportRoot">导出: </label>
          <input type="checkbox" v-model="exportRoot" :true-value="true" :false-value="false" />
        </div>
        <div class="gap"></div>
        <button @click="copyCode(displayedInterfaceContent)">复制</button>
        <div class="gap"></div>
        <a target="_blank" href="https://github.com/Dunn-liu/jstots">
          <img src="./github.svg" width="28" height="28" />
        </a>
      </div>
      <textarea class="preview-content" v-model="displayedInterfaceContent"></textarea>
    </div>
  </div>
</template>

<script>
import { computed, defineComponent, onMounted, ref, watch } from "vue";
import { transform, copyTextToClipboard } from "./util";

const inputTypes = {
  JS: "JS",
  JSON: "JSON",
  INVALID: "INVALID",
};

export default defineComponent({
  name: "App",
  setup() {
    const jsonContent = ref(``);
    const interfaceContent = ref("");
    const displayedInterfaceContent = ref("");
    const rootInterfaceName = ref("Test");
    const exportRoot = ref(true);
    const indentNumber = ref(2);
    const userInputType = ref(inputTypes.JSON);
    const inputRef = ref(null);

    const config = computed(() => {
      return {
        outputIndent: indentNumber.value,
        exportRoot: exportRoot.value,
        rootName: rootInterfaceName.value,
      };
    });

    const getTransformedCode = () => {
      displayedInterfaceContent.value = interfaceContent.value;
    };

    const handleContentOrConfigChange = () => {
      try {
        const parsedJson = JSON.parse(jsonContent.value);
        if (parsedJson) {
          userInputType.value = inputTypes.JSON;
          interfaceContent.value = transform(parsedJson, config.value, true);
        }
      } catch (error) {
        try {
          const jsObject = eval(`(${jsonContent.value})`);
          if (jsObject) {
            userInputType.value = inputTypes.JS;
            interfaceContent.value = transform(jsObject, config.value, true);
          }
        } catch (error) {
          userInputType.value = inputTypes.INVALID;
          interfaceContent.value = `Error: Invalid JSON or JS object`;
        }
      }
    };

    watch(
      () => config.value,
      () => {
        handleContentOrConfigChange();
        getTransformedCode();
      }
    );

    watch(() => jsonContent.value, handleContentOrConfigChange);

    onMounted(() => {
      setTimeout(() => {
        inputRef.value?.focus();
      }, 100);
    });

    function format() {
      if (userInputType.value === inputTypes.JS) {
        const jsObject = eval(`(${jsonContent.value})`);
        jsonContent.value = JSON.stringify(jsObject, null, 2);
      } else if (userInputType.value === inputTypes.JSON) {
        jsonContent.value = JSON.stringify(JSON.parse(jsonContent.value), null, 2);
      } else {
        alert(`Can't format invalid JSON or JS object`);
      }
    }

    return {
      inputRef,
      jsonContent,
      displayedInterfaceContent,
      format,
      rootInterfaceName,
      indentNumber,
      exportRoot,
      getTransformedCode,
      copyCode: copyTextToClipboard,
    };
  },
});
</script>

<style lang="css" scoped>
.container {
  display: flex;
  height: 100%;
}

.edit-container {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.edit-container .eidtor {
  flex: 1;
}

.gap {
  width: 1px;
  background: #eaeaea;
  height: 30px;
  margin: 0 12px;
}

.config-bar {
  display: flex;
  align-items: center;
  padding: 12px;
  border-bottom: 1px solid #eaeaea;
  position: relative;
}

.transform-btn {
  position: absolute;
  right: 0;
  transform: translateX(50%);
  z-index: 2;
  background-color: #0a66c2;
  background-image: none;
  padding: 4px 20px;
}

.transform-btn:hover {
  background-color: #16437e;
}

.preview-container .config-bar {
  justify-content: flex-end;
}

.preview-container {
  flex: 1;
}

.preview-content {
  padding: 24px;
  min-width: 30vw;
  border-left: 1px solid #eaeaea;
}

.input-control {
  height: 30px;
  display: flex;
  align-items: center;
}

label {
  margin-right: 4px;
}

textarea {
  min-height: 80vh;
  border: none;
  outline: none;
  resize: none;
  padding: 24px;
}

a {
  display: flex;
}
</style>
