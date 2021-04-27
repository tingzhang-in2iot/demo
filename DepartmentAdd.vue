<template>
  <div class="bg">
    <a-button type="dashed" @click="goback" style="margin: 20px; float: left"> 返回 </a-button>
    <a-layout style="background-color: #fff; padding: 16px 0">
      <a-form :form="form" style="text-align: left">
        <a-form-item>
          <a-button type="primary" html-type="submit" @click="addRole"> 保存 </a-button>
        </a-form-item>
        <a-row :gutter="[16]">
          <a-col :span="8" style="padding: 10px 15px">
            <a-row>
              <a-col :span="10">
                <!-- 账号account -->
                <a-form-item label="上级名称" :labelCol="{ span: 11 }" :wrapperCol="{ span: 13 }">
                  <a-input disabled class="input" v-decorator="['parentName']" key="parentName" />
                </a-form-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col :span="10">
                <a-form-item label="名称" :labelCol="{ span: 11 }" :wrapperCol="{ span: 13 }">
                  <a-input v-decorator="['name']" placeholder="请输入名称" />
                </a-form-item>
              </a-col>
              <a-col :span="14">
                <!-- 类型 -->
                <a-form-item label="人数" :labelCol="{ span: 8 }" :wrapperCol="{ span: 10 }">
                  <a-input v-decorator="['headCount']" suffix="人" placeholder="请输入人数" />
                </a-form-item>
              </a-col>
            </a-row>
          </a-col>
          <a-col :span="16" style="padding: 0 25px">
            <div style="margin: 10px 0">选择关联建筑：</div>
            <a-row style="width: 100%">
              <a-col>
                <a-transfer
                  :titles="['未选择建筑', '已选择建筑']"
                  :data-source="plainOptionsFlatten"
                  :list-style="{
                    width: '24%',
                    height: '65vh',
                    'overflow-y': 'auto'
                  }"
                  :target-keys="targetKeys"
                  :render="(item) => item.title"
                  @change="handleChange"
                >
                  <template slot="children" slot-scope="{ props: { direction, selectedKeys }, on: { itemSelect } }">
                    <a-tree
                      blockNode
                      checkStrictly
                      checkable
                      v-if="direction === 'left'"
                      :checkedKeys="[...selectedKeys, ...targetKeys]"
                      :treeData="plainOptions"
                      :replaceFields="{ key: 'id', title: 'name' }"
                      @check="
                        (_, props) => {
                          onChecked(_, props, [...selectedKeys, ...targetKeys], itemSelect)
                        }
                      "
                      @select="
                        (_, props) => {
                          onChecked(_, props, [...selectedKeys, ...targetKeys], itemSelect)
                        }
                      "
                    />
                  </template>
                </a-transfer>
              </a-col>
            </a-row>
          </a-col>
        </a-row>
      </a-form>
    </a-layout>
  </div>
</template>
<script lang="ts">
import Vue from 'vue'
import * as API from '@/service/settings/department'
import * as BUILDING_API from '@/service/settings/building'
import { WrappedFormUtils } from 'ant-design-vue/types/form/form'
import { Watch, Component } from 'vue-property-decorator'
import { getNode } from '@/utils/tree-util'
import { hideLoading, showLoading, tellNegative, tellPositive } from '@/utils/message-util'
import { StatusCode } from '@/config/enum'

function isChecked(selectedKeys: string | any[], eventKey: any) {
  return selectedKeys.includes(eventKey)
}

@Component
export default class Add extends Vue {
  targetKeys: string[] = []
  plainOptions: TreeData[] = []
  plainOptionsFlatten!: { key: string; title: string }[]
  form!: WrappedFormUtils
  selectedKeys: string[] = []
  templateCheckedKeys: string[] = []

  @Watch('selectedKeys')
  handleChange(targetKeys: string[]) {
    this.targetKeys = targetKeys
    this.plainOptions = this.handleTreeData(this.plainOptions, this.targetKeys)
  }

  goback(): void {
    this.$router.go(-1)
  }

  flatten(list: TreeData[]) {
    list.forEach((item) => {
      this.plainOptionsFlatten.push({ key: item.id, title: item.name })
      if (item.children) {
        this.flatten(item.children)
      }
    })
  }

  handleTreeData(data: TreeData[], targetKeys: string[] = []) {
    data.forEach((item) => {
      item['disabled'] = targetKeys.includes(item.id)
      if (item.children) {
        this.handleTreeData(item.children, targetKeys)
      }
    })
    return data
  }

  onChecked(_: any, e: { node: { eventKey: any } }, checkedKeys: string | any[], itemSelect: (arg0: any, arg1: boolean) => void) {
    const { eventKey } = e.node
    itemSelect(eventKey, !isChecked(checkedKeys, eventKey))
  }

  /**
   * 新增
   */
  addRole(): void {
    this.form.validateFields(async (err, values) => {
      const response = await API.addDepartment(
        {
          parentId: this.$store.getters.getDepartmentParentId,
          name: values.name,
          headCount: Number(values.headCount)
        },
        this.targetKeys.filter((tk) => !getNode(this.plainOptions[0], tk)?.isRoom),
        this.targetKeys.filter((tk) => getNode(this.plainOptions[0], tk)?.isRoom)
      )
      if (response.status === StatusCode.SUCCESS) {
        this.$router.go(-1)
        tellPositive('部门已新增！')
      } else {
        tellNegative(response.message)
      }
    })
  }

  beforeCreate() {
    this.form = this.$form.createForm(this)
  }

  async created() {
    showLoading('查询建筑中，请稍等..')
    this.plainOptions = (await BUILDING_API.getPlumpBuildings()).data
    this.plainOptionsFlatten = []
    this.flatten(this.plainOptions)
    this.targetKeys = []
    const data = (await API.getDepartmentInfo(this.$store.getters.getDepartmentParentId)).data
    this.form.setFieldsValue({ parentName: data.name })
    hideLoading()
  }
}
</script>
<style scoped>
.bg {
  background-color: white;
  height: 92vh;
}
.input {
  float: left;
  display: flex;
  justify-content: center;
}
.show:empty:before {
  content: attr(placeholder);
}
.add_left {
  position: absolute;
  width: 45%;
  height: 80vh;
  left: 0;
}
</style>
