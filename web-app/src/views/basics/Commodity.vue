<template>
  <div>
    <div style="display: flex">
      <a-button size="large" class="editable-add-btn" @click="commodityVisible = true">
        <a-icon type="plus"/>
        新增商品
      </a-button>
      <a-input-search
          placeholder="请输入商品名"
          enter-button="搜索商品"
          style="width: 400px;margin-left: 20px"
          size="large"
          @search="onSearch"
      />
      <a-button style="margin-left: 10px" size="large" type="danger" @click="loadTableData">
        重置
      </a-button>
    </div>
    <a-table :loading="loading" :columns="columns" :data-source="data" :sortDirections="['ascend', 'descend']" rowKey="id">
      <a slot="name" slot-scope="text">{{ text }}</a>
      <span slot="customTitle"><a-icon type="smile-o"/> 商品名称</span>
      <span slot="action" slot-scope="text, record, index">
        <a-button @click="handleUpdate(record)" type="link"><a-icon type="edit"/> Update</a-button>
      <a-divider type="vertical"/>
         <a-button @click="handleDelete(record,index)" type="link"><a-icon type="delete"/> Delete</a-button>
      </span>
    </a-table>

    <a-modal
        title="商品信息"
        :closable="false"
        :visible="commodityVisible"
    >
      <a-form-model ref="ruleForm" :model="commodity">
        <a-form-model-item ref="name" label="商品名称" prop="name">
          <a-input v-model="commodity.name"/>
        </a-form-model-item>
        <a-form-model-item label="商品单价" prop="price">
          <a-input-number id="input" v-model="commodity.price" :min="1"/>
        </a-form-model-item>
        <a-form-model-item label="描述信息" prop="description">
          <a-input v-model="commodity.description" type="textarea"/>
        </a-form-model-item>
      </a-form-model>
      <template slot="footer">
        <a-button key="back" @click="commodityVisible = false">
          Return
        </a-button>
        <a-button key="submit" type="primary" :loading="modalLoading" @click="submitCommodity">
          Submit
        </a-button>
      </template>
    </a-modal>

  </div>
</template>

<script>
import {DeleteCommodityById, FindAllCommodity, SaveCommodity} from "@/api/commodity";
import {SearchCommodity} from "../../api/commodity";

const columns = [
  {
    dataIndex: 'name',
    key: 'name',
    slots: {title: 'customTitle'},
    scopedSlots: {customRender: 'name'},
  },
  {
    title: '库存数量',
    dataIndex: 'count',
    key: 'age'
    // defaultSortOrder: 'descend', // 默认上到下为由大到小的顺序
    // sorter: (a, b) => { return a.count> b.count? 1 : -1 },
    // sortField: 'count'
  },
  {
    title: '描述信息',
    dataIndex: 'description',
    key: 'description',
  },
  {
    title: '入库时间',
    dataIndex: 'createAt',
    key: 'createAt',
    defaultSortOrder: 'descend', // 默认上到下为由大到小的顺序
    sorter: (a, b) => { return a.createAt> b.createAt? 1 : -1 },
    sortField: 'createAt'
  },
  {
    title: '商品单价',
    key: 'price',
    dataIndex: 'price',
    scopedSlots: {customRender: 'tags'},
  },
  {
    title: '更多操作',
    key: 'action',
    scopedSlots: {customRender: 'action'},
  }
];

const data = [];

export default {
  data() {
    return {
      commodity: {
        name: '',
        description: '商品简介',
        count: 0,
        price: 9.99,
      },
      loading: false,
      modalLoading: false,
      commodityVisible: false,
      commodityLoading: false,
      data: [],
      columns,
    };
  },

  mounted() {
    this.loadTableData()
  },

  methods: {
    onSearch(value) {
      if (value){
        this.loading = true
        SearchCommodity(value).then((res) => {
          console.log(res)
          if (res.data.length === 0) {
            this.$message.warn("未搜索到任何数据")
            setTimeout(() => {
              this.loading = false
              this.data = res.data
            }, 600)
          } else {
            setTimeout(() => {
              this.$message.success("搜索到" + res.data.length + "个商品")
              this.loading = false
              this.data = res.data
            }, 600)
          }
        })
      }else {
        this.$message.warn("请输入搜索内容")
      }
    },

    loadTableData() {
      this.loading = true
      FindAllCommodity().then((res) => {
        setTimeout(() => {
          this.loading = false
          this.data = res.data
        }, 600)
      })
    },
    submitCommodity() {
      this.modalLoading = true
      SaveCommodity(this.commodity).then((res) => {
        if (res.status) {
          setTimeout(() => {
            this.modalLoading = false
            this.commodityVisible = false
            this.$message.success('商品信息提交成功');
            this.loadTableData()
          }, 600)
        } else {
          setTimeout(() => {
            this.modalLoading = false
          }, 600)
        }
      })
    },

    handleDelete(r, index) {
      DeleteCommodityById(r.id).then((res) => {
        if (res.status) this.$message.success('商品信息删除成功');
        this.loadTableData()
      })
      console.log(index)
    },

    handleUpdate(r) {
      this.commodity = r
      this.commodityVisible = true
    },

  },
};
</script>

<style scoped>
.editable-add-btn {
  margin-bottom: 15px;
}
</style>
