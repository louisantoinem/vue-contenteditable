
<style scoped>

</style>

<template>
  <component
    :is="tag"
    :contenteditable="contenteditable"
    @input="update"
    @blur="update"
    @paste="onPaste"
    @keypress="onKeypress"
    ref="element"
  >
  </component>
</template>

<script lang="ts">


import { defineComponent, ref, onMounted, watch } from 'vue';

function replaceAll(str: string, search: string, replacement: string) {
  return str.split(search).join(replacement);
}

export default defineComponent({
  name : 'contenteditable',
  props : {
    'tag' : String,
    'contenteditable' : {
      type : Boolean,
      default : true,
    },
    'modelValue' : String,
    'noHTML' : {
      type : Boolean,
      default : true,
    },
    'interceptEnter' : {
      type : Boolean,
      default : false,
    },
  },
  emits : {
    'returned' : String,
    'update:modelValue' : String,
  },
  setup(props, {emit} ){
    const element = ref<HTMLElement | null>(null)

    function currentContent(){
      return props.noHTML ?
        element.value!.innerText
        :
        element.value!.innerHTML
    }
    function updateContent(newcontent: string){
      if(props.noHTML) {
        element.value!.innerText = newcontent;
      }
      else {
        element.value!.innerHTML = newcontent;
      }
    }
    function update() {
      emit('update:modelValue', currentContent());
    }
    function onPaste(event: any) {
      event.preventDefault();
      const text = (event.originalEvent || event).clipboardData.getData('text/plain');
      window.document.execCommand('insertText', false, text);
    }
    function onKeypress(event: any) {
      if(event.key == 'Enter' && !event.shiftKey && props.interceptEnter) {
        event.preventDefault();
        emit('returned', currentContent());
      }
    }
    onMounted(() =>{
      updateContent(props.modelValue ?? '')
    })
    watch( () => props.modelValue, (newval, oldval) => {
      if(newval != currentContent()){
        updateContent(newval ?? '')
      }
    })
    return {
      update,
      onPaste,
      onKeypress,
      element,
    }
  },
})
</script>
