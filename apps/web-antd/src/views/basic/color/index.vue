<script lang="ts" setup>
import type { VxeGridProps } from '#/adapter/vxe-table';
import type { ColorApi } from '#/api';

import { h, reactive } from 'vue';

import { Page, useVbenModal } from '@vben/common-ui';

import { Button, InputSearch, Modal } from 'ant-design-vue';

import { useVbenVxeGrid } from '#/adapter/vxe-table';
import { deleteColorApi, getColorListApi } from '#/api';

import ExtraModal from './modal.vue';

defineOptions({ name: 'Color' });
const form = reactive({
  keyword: undefined,
});
const gridOptions: VxeGridProps = {
  columns: [
    {
      title: '编码',
      field: 'id',
      width: 70,
    },
    {
      title: '名称',
      field: 'name',
      width: 220,
    },
    {
      title: '排序',
      field: 'sort',
      width: 100,
    },
    {
      title: '创建时间',
      field: 'createdAt',
      width: 145,
    },
    {
      title: '更新时间',
      field: 'updatedAt',
      width: 145,
    },
    {
      title: '操作',
      field: 'action',
      width: 120,
      slots: {
        default: 'action',
      },
      resizable: false,
    },
  ],
  proxyConfig: {
    ajax: {
      // , sort
      query: async ({ page }) => {
        return await getColorListApi({
          pageNum: page.currentPage,
          pageSize: page.pageSize,
          ...form,
          // sortBy: sort.field,
          // sortOrder: sort.order,
        });
      },
    },
    // sort: true,
  },
  // sortConfig: {
  //   defaultSort: { field: 'id', order: 'desc' },
  //   remote: true,
  // },
};
const gridEvents: VxeGridListeners<ColorApi.ColorItem> = {
  cellDblclick: ({ row }) => handleEdit(row),
};
const [Grid, gridApi] = useVbenVxeGrid({ gridOptions, gridEvents });
const [ColorModal, modalApi] = useVbenModal({
  connectedComponent: ExtraModal,
  destroyOnClose: true,
});
const handleAdd = () => {
  modalApi.setData({}).open();
};
const handleEdit = (row: ColorApi.ColorItem) => {
  modalApi.setData(row).open();
};
const handleDelete = (row: ColorApi.ColorItem) => {
  Modal.confirm({
    icon: null,
    centered: true,
    maskClosable: false,
    destroyOnClose: true,
    title: '删除颜色',
    content: h('div', null, [
      h('span', null, '您确定要删除 '),
      h('span', { class: 'text-red-500' }, `${row.name}`),
      h('span', null, ' 吗 ?'),
    ]),
    async onOk() {
      try {
        await deleteColorApi(row.id);
        gridApi.query();
      } catch {}
    },
  });
};
const handleRefresh = (code: 'query' | 'reload') => {
  if (code === 'query') {
    gridApi.query();
  } else {
    gridApi.reload();
  }
};
</script>
<template>
  <Page auto-content-height>
    <template #title>
      <div class="flex items-center justify-between">
        <div>
          <InputSearch
            allow-clear
            enter-button="查询"
            v-model:value="form.keyword"
            class="flex-1"
            placeholder="请输入颜色编码、名称"
            @search="handleRefresh('reload')"
          />
        </div>
        <div>
          <Button type="primary" @click="handleAdd">新增</Button>
        </div>
      </div>
    </template>
    <Grid>
      <template #action="{ row }">
        <Button type="link" size="small" @click="handleEdit(row)">
          修改
        </Button>
        <Button type="link" size="small" danger @click="handleDelete(row)">
          删除
        </Button>
      </template>
    </Grid>
    <ColorModal @refresh="handleRefresh" />
  </Page>
</template>
