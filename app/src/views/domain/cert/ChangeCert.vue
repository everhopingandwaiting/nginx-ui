<script setup lang="tsx">
import { Badge } from 'ant-design-vue'
import type { ComputedRef, Ref } from 'vue'
import StdTable from '@/components/StdDesign/StdDataDisplay/StdTable.vue'
import type { Cert } from '@/api/cert'
import cert from '@/api/cert'
import type { customRender } from '@/components/StdDesign/StdDataDisplay/StdTableTransformer'
import { input } from '@/components/StdDesign/StdDataEntry'
import type { NgxDirective } from '@/api/ngx'
import type { Column, JSXElements } from '@/components/StdDesign/types'

const current_server_directives = inject('current_server_directives') as ComputedRef<NgxDirective[]>
const directivesMap = inject('directivesMap') as Ref<Record<string, NgxDirective[]>>
const visible = ref(false)

const columns: Column[] = [{
  title: () => $gettext('Name'),
  dataIndex: 'name',
  sortable: true,
  pithy: true,
  customRender: (args: customRender) => {
    const { text, record: r } = args
    if (!text)
      return h('div', r.domain)

    return h('div', text)
  },
  edit: {
    type: input,
  },
  search: true,
}, {
  title: () => $gettext('Auto Cert'),
  dataIndex: 'auto_cert',
  customRender: (args: customRender) => {
    const template: JSXElements = []
    const { text } = args
    if (text === true || text > 0) {
      template.push(<Badge status="success"/>)
      template.push($gettext('Enabled'))
    }
    else {
      template.push(<Badge status="warning"/>)
      template.push($gettext('Disabled'))
    }

    return h('div', template)
  },
  sortable: true,
  pithy: true,
}]

function open() {
  visible.value = true
}

const records = ref([]) as Ref<Cert[]>

function ok() {
  if (directivesMap.value.ssl_certificate?.[0]) {
    directivesMap.value.ssl_certificate[0].params = records.value[0].ssl_certificate_path
  }
  else {
    current_server_directives?.value.push({
      directive: 'ssl_certificate',
      params: records.value[0].ssl_certificate_path,
    })
  }
  if (directivesMap.value.ssl_certificate_key?.[0]) {
    directivesMap.value.ssl_certificate_key[0].params = records.value[0].ssl_certificate_key_path
  }
  else {
    current_server_directives?.value.push({
      directive: 'ssl_certificate_key',
      params: records.value[0].ssl_certificate_key_path,
    })
  }
  visible.value = false
}
const selectedKeyBuffer = ref({})

const computedSelectedKeys = computed({
  get() {
    return [selectedKeyBuffer.value]
  },
  set(v) {
    selectedKeyBuffer.value = v
  },
})
</script>

<template>
  <div>
    <AButton @click="open">
      {{ $gettext('Change Certificate') }}
    </AButton>
    <AModal
      v-model:open="visible"
      :title="$gettext('Change Certificate')"
      :mask="false"
      @ok="ok"
    >
      <StdTable
        v-model:selected-row-keys="computedSelectedKeys"
        v-model:selected-rows="records"
        :api="cert"
        pithy
        :columns="columns"
        selection-type="radio"
      />
    </AModal>
  </div>
</template>

<style lang="less" scoped>

</style>
