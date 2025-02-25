<template>
  <a-spin :loading="loading">
    <a-form
      ref="formRef"
      :model="form"
      :rules="form.STORAGE_DEFAULT === 'LOCAL' ? localRules : ossRules"
      auto-label-width
      label-align="left"
      :layout="width >= 500 ? 'horizontal' : 'vertical'"
      :disabled="!isUpdate"
      scroll-to-first-error
      class="form"
    >
      <!-- 默认存储 -->
      <a-form-item
        field="STORAGE_DEFAULT"
        :label="storageConfig.STORAGE_DEFAULT.name"
      >
        <a-radio-group v-model="form.STORAGE_DEFAULT">
          <a-radio value="LOCAL">本地存储</a-radio>
          <a-radio value="OSS">对象存储</a-radio>
        </a-radio-group>
      </a-form-item>

      <fieldset>
        <legend>本地存储配置</legend>
        <a-form-item
          field="STORAGE_LOCAL_BUCKET"
          :label="storageConfig.STORAGE_LOCAL_BUCKET.name"
          :help="storageConfig.STORAGE_LOCAL_BUCKET.description"
          hide-asterisk
        >
          <a-input v-model="form.STORAGE_LOCAL_BUCKET" />
        </a-form-item>
        <a-form-item
          field="STORAGE_LOCAL_ENDPOINT"
          :label="storageConfig.STORAGE_LOCAL_ENDPOINT.name"
          :help="storageConfig.STORAGE_LOCAL_ENDPOINT.description"
          hide-asterisk
        >
          <a-input v-model="form.STORAGE_LOCAL_ENDPOINT" />
        </a-form-item>
      </fieldset>

      <fieldset>
        <legend>对象存储配置</legend>
        <a-form-item
          field="STORAGE_OSS_ACCESS_KEY"
          :label="storageConfig.STORAGE_OSS_ACCESS_KEY.name"
          :help="storageConfig.STORAGE_OSS_ACCESS_KEY.description"
          hide-asterisk
        >
          <a-input v-model="form.STORAGE_OSS_ACCESS_KEY" />
        </a-form-item>
        <a-form-item
          field="STORAGE_OSS_SECRET_KEY"
          :label="storageConfig.STORAGE_OSS_SECRET_KEY.name"
          :help="storageConfig.STORAGE_OSS_SECRET_KEY.description"
          hide-asterisk
        >
          <a-input-password v-model="form.STORAGE_OSS_SECRET_KEY" />
        </a-form-item>
        <a-form-item
          field="STORAGE_OSS_BUCKET"
          :label="storageConfig.STORAGE_OSS_BUCKET.name"
          :help="storageConfig.STORAGE_OSS_BUCKET.description"
          hide-asterisk
        >
          <a-input v-model="form.STORAGE_OSS_BUCKET" />
        </a-form-item>
        <a-form-item
          field="STORAGE_OSS_ENDPOINT"
          :label="storageConfig.STORAGE_OSS_ENDPOINT.name"
          :help="storageConfig.STORAGE_OSS_ENDPOINT.description"
          hide-asterisk
        >
          <a-input v-model="form.STORAGE_OSS_ENDPOINT" />
        </a-form-item>
        <a-form-item
          field="STORAGE_OSS_REGION"
          :label="storageConfig.STORAGE_OSS_REGION.name"
          :help="storageConfig.STORAGE_OSS_REGION.description"
          hide-asterisk
        >
          <a-input v-model="form.STORAGE_OSS_REGION" />
        </a-form-item>
      </fieldset>

      <!-- 操作按钮 -->
      <a-space style="margin-bottom: 16px">
        <a-button v-if="!isUpdate" v-permission="['system:config:update']" type="primary" @click="onUpdate">
          <template #icon>
            <icon-edit />
          </template>
          修改
        </a-button>
        <a-button v-if="!isUpdate" v-permission="['system:config:reset']" @click="onResetValue">
          <template #icon>
            <icon-undo />
          </template>
          恢复默认
        </a-button>
        <a-button v-if="isUpdate" type="primary" @click="handleSave">
          <template #icon>
            <icon-save />
          </template>
          保存
        </a-button>
        <a-button v-if="isUpdate" @click="reset">
          <template #icon>
            <icon-refresh />
          </template>
          重置
        </a-button>
        <a-button v-if="isUpdate" @click="handleCancel">
          <template #icon>
            <icon-undo />
          </template>
          取消
        </a-button>
      </a-space>
    </a-form>
  </a-spin>
</template>

<script setup lang="ts">
import { useWindowSize } from '@vueuse/core'
import { type FormInstance, Message, Modal } from '@arco-design/web-vue'
import {
  type OptionResp,
  type SecurityConfig,
  type StorageConfig,
  listOption,
  resetOptionValue,
  updateOption,
} from '@/apis/system'
import { useResetReactive } from '@/hooks'

defineOptions({ name: 'StorageSetting' })
const { width } = useWindowSize()

const loading = ref<boolean>(false)
const formRef = ref<FormInstance>()

const [form] = useResetReactive({
  STORAGE_DEFAULT: 'LOCAL', // 默认选中本地存储
  STORAGE_LOCAL_BUCKET: '',
  STORAGE_LOCAL_ENDPOINT: '',
  STORAGE_OSS_ACCESS_KEY: '',
  STORAGE_OSS_SECRET_KEY: '',
  STORAGE_OSS_BUCKET: '',
  STORAGE_OSS_ENDPOINT: '',
  STORAGE_OSS_REGION: '',
})

const localRules: FormInstance['rules'] = {
  STORAGE_LOCAL_BUCKET: [{ required: true, message: '请输入本地存储路径' }],
  STORAGE_LOCAL_ENDPOINT: [{ required: true, message: '请输入本地资源访问地址' }],
}

const ossRules: FormInstance['rules'] = {
  STORAGE_OSS_ACCESS_KEY: [{ required: true, message: '请输入Access Key' }],
  STORAGE_OSS_SECRET_KEY: [{ required: true, message: '请输入Secret Key' }],
  STORAGE_OSS_BUCKET: [{ required: true, message: '请输入对象存储桶名称' }],
  STORAGE_OSS_ENDPOINT: [{ required: true, message: '请输入对象存储终端节点' }],
}

const storageConfig = ref<StorageConfig>({
  STORAGE_DEFAULT: {},
  STORAGE_LOCAL_BUCKET: {},
  STORAGE_LOCAL_ENDPOINT: {},
  STORAGE_OSS_ACCESS_KEY: {},
  STORAGE_OSS_SECRET_KEY: {},
  STORAGE_OSS_BUCKET: {},
  STORAGE_OSS_ENDPOINT: {},
  STORAGE_OSS_REGION: {},
})

const reset = () => {
  formRef.value?.resetFields()
  Object.keys(form).forEach((key) => {
    form[key] = storageConfig.value[key]?.value || form[key] // 兜底设置默认值
  })
}

const isUpdate = ref(false)

const onUpdate = () => {
  isUpdate.value = true
}

const handleCancel = () => {
  reset()
  isUpdate.value = false
}

const queryForm = { category: 'STORAGE' }

const getDataList = async () => {
  loading.value = true
  const { data } = await listOption(queryForm)
  storageConfig.value = data.reduce((obj: SecurityConfig, option: OptionResp) => {
    obj[option.code] = { ...option, value: option.value }
    return obj
  }, {})
  handleCancel()
  loading.value = false
}

const handleSave = async () => {
  const isInvalid = await formRef.value?.validate()
  if (isInvalid) return
  await updateOption(
    Object.entries(form).map(([key, value]) => ({ id: storageConfig.value[key].id, code: key, value })),
  )
  await getDataList()
  Message.success('保存成功')
}

const handleResetValue = async () => {
  await resetOptionValue(queryForm)
  Message.success('恢复成功')
  await getDataList()
}

const onResetValue = () => {
  Modal.warning({
    title: '警告',
    content: '确认恢复存储配置为默认值吗？',
    hideCancel: false,
    maskClosable: false,
    onOk: handleResetValue,
  })
}

onMounted(() => {
  getDataList()
})
</script>

<style scoped lang="scss">
:deep(.form .arco-input-wrapper) {
  width: 400px;
}

fieldset {
  display: inline-block;
  width: fit-content;
  min-width: 0;
  padding: 15px;
  margin-bottom: 15px;
  border: 1px solid var(--color-neutral-3);
  border-radius: 3px;
}
fieldset legend {
  color: rgb(var(--gray-10));
  padding: 2px 5px 2px 5px;
  border: 1px solid var(--color-neutral-3);
  border-radius: 3px;
}
</style>
