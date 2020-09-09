<template>
  <div class="apos-area__outer">
    <AposAreaEditor 
      :doc-id="docId"
      :doc-type="docType"
      :id="id"
      :field-id="fieldId"
      :options="options"
      :items="items"
      :choices="choices"
      @supress="supress"
    />
  </div>
</template>

<script>
export default {
  props: {
    docId: {
      type: String,
      default: null
    },
    docType: {
      type: String,
      default: null
    },
    id: {
      type: String,
      required: true
    },
    fieldId: {
      type: String,
      required: true
    },
    options: {
      type: Object,
      default() {
        return {};
      }
    },
    items: {
      type: Array,
      default() {
        return [];
      }
    },
    choices: {
      type: Array,
      required: true
    }
  },
  methods: {
    supress() {
      console.log('hellooooo?!?!?!!3232');
    }
  }
}
</script>