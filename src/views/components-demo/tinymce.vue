<template>
  <div class="components-container">
    <div>
      <tinymce v-model="content" :height="300" />
    </div>
  </div>
</template>

<script>
import Tinymce from '@/components/Tinymce'

export default {
  name: 'TinymceDemo',
  components: { Tinymce },
  data() {
    return {
      content:"aaa",
    }
  }
}
</script>

<style scoped>
.editor-content{
  margin-top: 20px;
}
</style>

