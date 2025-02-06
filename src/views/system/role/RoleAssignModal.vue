<template>
  <a-modal
    v-model:visible="visible"
    title="分配角色"
    :mask-closable="false"
    :esc-to-close="false"
    :width="width >= 1100 ? 1100 : '100%'"
    draggable
    @before-ok="save"
    @close="reset"
  >
    <UserSelect v-if="visible" ref="UserSelectRef" v-model:value="selectedUsers" :exclude-value="excludeUsers" @select-user="onSelectUser" />
  </a-modal>
</template>

<script setup lang="ts">
import { Message } from '@arco-design/web-vue'
import { useWindowSize } from '@vueuse/core'
import { assignToUsers, listRoleUserId } from '@/apis/system/role'

const emit = defineEmits<{
  (e: 'save-success'): void
}>()

const { width } = useWindowSize()

const dataId = ref('')
const visible = ref(false)
const selectedUsers = ref<string[]>([])
const excludeUsers = ref<string[]>([])

// 用户选择回调
const onSelectUser = (value: string[]) => {
  selectedUsers.value = value
}

const UserSelectRef = ref()
// 重置
const reset = () => {
  dataId.value = ''
  selectedUsers.value = []
  UserSelectRef.value?.onClearSelected()
}

// 保存
const save = async () => {
  try {
    const isInvalid = selectedUsers.value.length === 0
    if (isInvalid) {
      Message.warning('请选择用户')
      return false
    }
    await assignToUsers(dataId.value, selectedUsers.value)
    Message.success('分配成功')
    reset()
    emit('save-success')
    return true
  } catch (error) {
    return false
  }
}

// 打开
const onOpen = async (id: string) => {
  dataId.value = id
  // 初始化选择的用户
  const { data } = await listRoleUserId(id)
  excludeUsers.value = data
  selectedUsers.value = []
  visible.value = true
}

defineExpose({ onOpen })
</script>

<style scoped lang="scss"></style>
