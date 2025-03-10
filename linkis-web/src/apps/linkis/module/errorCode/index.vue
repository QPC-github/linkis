<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one or more
  ~ contributor license agreements.  See the NOTICE file distributed with
  ~ this work for additional information regarding copyright ownership.
  ~ The ASF licenses this file to You under the Apache License, Version 2.0
  ~ (the "License"); you may not use this file except in compliance with
  ~ the License.  You may obtain a copy of the License at
  ~
  ~   http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
-->

<template>
  <div :style="{height: '100%', overflow: 'hidden'}">
    <Row class="search-bar" type="flex">
      <Col span="6">
        <span :style="{ whiteSpace: 'nowrap', marginRight: '5px', fontSize: '14px', lineHeight: '32px'}" :title="$t('message.linkis.basedataManagement.searchLabel')">{{$t('message.linkis.basedataManagement.searchLabel')}}</span>
        <Input v-model="searchName" clearable suffix="ios-search" class="input" :placeholder="$t('message.linkis.basedataManagement.errorCode.searchPlaceholder')"></Input>
      </Col>
      <Col span="3">
        <Button type="primary" class="Button" @click="load()">{{
          $t('message.linkis.basedataManagement.search')
        }}
        </Button>
        <Button type="success" class="Button" style="margin-left: 10px" @click="onAdd()">{{
          $t('message.linkis.basedataManagement.add')
        }}
        </Button>
      </Col>
      <Col span="15">
      </Col>
    </Row>
    <Table border size="small" align="center" :columns="tableColumnNum" :data="pageDatalist"
      class="table-content mytable">
      <template slot-scope="{ row,index }" slot="action">
        <ButtonGroup size="small">
          <Button
            :disabled="row.expire"
            size="small"
            type="primary"
            @click="onTableEdit(row, index)"
          >{{ $t('message.linkis.basedataManagement.edit') }}
          </Button
          >
          <Button
            :disabled="row.expire"
            size="small"
            type="primary"
            @click="onTableDelete(row, index)"
          >
            {{ $t('message.linkis.basedataManagement.remove') }}
          </Button>
        </ButtonGroup>
      </template>
    </Table>
    <div style="margin: 10px; overflow: hidden; textAlign: center">
      <div>
        <Page
          :page-size="page.pageSize"
          :total="page.totalSize"
          :current="page.pageNow"
          @on-change="changePage"
          size="small"
          show-total
          show-elevator
          :prev-text="$t('message.linkis.previousPage')" :next-text="$t('message.linkis.nextPage')"
        ></Page>
      </div>
    </div>
    <Modal
      width="800"
      class="modal"
      v-model="modalShow"
      :title="modalAddMode=='add'? $t('message.linkis.basedataManagement.add') : $t('message.linkis.basedataManagement.edit')"
      :loading="modalLoading"
    >
      <div slot="footer">
        <Button type="text" size="large" @click="onModalCancel()">{{$t('message.linkis.basedataManagement.modal.cancel')}}</Button>
        <Button type="primary" size="large" @click="onModalOk('userConfirm')">{{$t('message.linkis.basedataManagement.modal.confirm')}}</Button>
      </div>
      <ErrorCodeForm ref="errorCodeForm" :data="modalEditData"></ErrorCodeForm>
    </Modal>
  </div>
</template>
<script>
import mixin from '@/common/service/mixin';
import ErrorCodeForm from './errorCodeForm/index'
import {add, del, edit, getList} from "./service";

export default {
  mixins: [mixin],
  components: {ErrorCodeForm},
  data() {
    return {
      searchName: "",
      page: {
        totalSize: 0,
        pageSize: 10,
        pageNow: 1,
      },
      tableColumnNum: [
        {
          title: "ID",
          key: 'id',
          width: 100,
          tooltip: true,
          align: 'center',
        },
        {
          title: this.$t('message.linkis.basedataManagement.errorCode.errorCode'),
          key: 'errorCode',
          width: 100,
          tooltip: true,
          align: 'center',
        },
        {
          title: this.$t('message.linkis.basedataManagement.errorCode.errorDesc'),
          key: 'errorDesc',
          tooltip: true,
          align: 'center',
        },
        {
          title: this.$t('message.linkis.basedataManagement.errorCode.errorRegex'),
          key: 'errorRegex',
          tooltip: true,
          align: 'center',
        },
        {
          title: this.$t('message.linkis.basedataManagement.action'),
          width: 150,
          slot: 'action',
          align: 'center',
        },
      ],
      pageDatalist: [],
      modalShow: false,
      modalAddMode: 'add',
      modalEditData: {},
      modalLoading: false
    };
  },
  created() {
    this.load()
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      window.console.log(this.$route.query.isSkip);
    },
    load() {
      let params = {
        searchName: this.searchName,
        currentPage: this.page.pageNow,
        pageSize: this.page.pageSize
      }
      getList(params).then((data) => {
        this.pageDatalist = data.list.list
        this.page.totalSize = data.list.total
      })
    },
    changePage(value) {
      this.page.pageNow = value
      this.load()
    },
    onAdd(){
      this.modalEditData={
        id: "",
        errorCode: "",
        errorDesc: "",
        errorRegex: '',
      }
      this.modalAddMode = 'add'
      this.modalShow = true
    },
    onTableEdit(row){
      this.modalEditData = row
      this.modalAddMode = 'edit'
      this.modalShow = true
    },
    onTableDelete(row){

      this.$Modal.confirm({
        title: this.$t('message.linkis.basedataManagement.modal.modalTitle'),
        content: this.$t('message.linkis.basedataManagement.modal.modalDelete', {name: row.errorCode}),
        onOk: ()=>{
          let params = {
            id: row.id
          }
          del(params).then((data)=>{
            if(data.result) {
              this.$Message.success({
                duration: 3,
                content: this.$t('message.linkis.basedataManagement.modal.modalDeleteSuccess')
              })
            }else{
              this.$Message.success({
                duration: 3,
                content: this.$t('message.linkis.basedataManagement.modal.modalDeleteFail')
              })
            }
            this.load()
          })
        }
      })

    },
    onModalOk(){
      this.$refs.errorCodeForm.formModel.submit((formData)=>{
        this.modalLoading = true
        if(this.modalAddMode=='add') {
          add(formData).then((data)=>{
            window.console.log(data)
            if(data.result) {
              this.$Message.success({
                duration: 3,
                content: this.$t('message.linkis.basedataManagement.modal.modalAddSuccess')
              })
            }else{
              this.$Message.success({
                duration: 3,
                content: this.$t('message.linkis.basedataManagement.modal.modalAddFail')
              })
            }
            this.load()
          })
        }else {
          edit(formData).then((data)=>{
            window.console.log(data)
            if(data.result) {
              this.$Message.success({
                duration: 3,
                content: this.$t('message.linkis.basedataManagement.modal.modalEditSuccess')
              })
              this.load()
            }else{
              this.$Message.success({
                duration: 3,
                content: this.$t('message.linkis.basedataManagement.modal.modalEditFail')
              })
            }
          })
          this.load()
        }
        this.modalLoading=false
        this.modalShow = false
      })
    },
    onModalCancel(){
      this.modalLoading=false
      this.modalShow = false
    }
  },
};
</script>

<style lang="scss" src="./index.scss" scoped>
</style>

<style lang="scss">
.mytable {
  border: 0;
  height: calc(100% - 110px);
  width: 100%;
  overflow-y: auto;

  .ivu-table:before {
    height: 0
  }

  .ivu-table:after {
    width: 0
  }

  .ivu-table {
    height: auto;
    border: 1px solid #dcdee2;
    width: 100%;
  }
}
</style>
