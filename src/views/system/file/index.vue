<template>
  <a-row align="stretch" :gutter="14" class="file-manage">
    <a-col :xs="0" :sm="8" :md="7" :lg="6" :xl="5" :xxl="4" flex="220px" class="h-full overflow-hidden">
      <FileAside></FileAside>
    </a-col>
    <a-col :xs="24" :sm="16" :md="17" :lg="18" :xl="19" :xxl="20" flex="1" class="h-full overflow-hidden">
      <FileMain></FileMain>
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import FileAside from './main/FileAside.vue'
import FileMain from './main/FileMain/index.vue'

defineOptions({ name: 'SystemFile' })
</script>

<style scoped lang="scss">
.file-manage {
  flex: 1;
  padding: $margin;
  overflow: hidden;
}
</style>
