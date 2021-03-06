<template>
  <div class="app-container">
    <el-card>
      <div slot="header" class="clearfix">
        <span>附件管理</span>
      </div>

      <div class="filter-container">
        <el-input v-model="listQuery.searchword" placeholder="请输入关键字" clearable style="width: 200px;margin-right: 10px;" class="filter-item" @keyup.enter.native="handleFilter" />         
        <el-select v-model="listQuery.status" placeholder="状态" clearable class="filter-item" style="width: 130px;margin-right: 10px;">
          <el-option v-for="item in statusOptions" :key="item.key" :label="item.display_name" :value="item.key" />
        </el-select>      
        <el-select v-model="listQuery.order" style="width: 140px;margin-right: 10px;" class="filter-item" @change="handleFilter">
          <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" />
        </el-select>
        <el-button v-waves class="filter-item" style="margin-right: 10px;" type="primary" icon="el-icon-search" @click="handleFilter">
          {{ $t('table.search') }}
        </el-button>
      </div>

      <el-table v-loading="listLoading" 
        ref="logTable"
        :header-cell-style="{background:'#eef1f6',color:'#606266'}"
        :data="list" border fit highlight-current-row 
        style="width: 100%">

        <el-table-column min-width="150px" label="文件名">
          <template slot-scope="{row}">
            <span>{{ row.name }}</span>
          </template>
        </el-table-column>

        <el-table-column width="110px" label="文件大小">
          <template slot-scope="{row}">
            <span>{{ row.size | renderSize }}</span>
          </template>
        </el-table-column>

        <el-table-column width="90px" label="文件类型">
          <template slot-scope="{row}">
            <span>{{ row.extension }}</span>
          </template>
        </el-table-column>

        <el-table-column width="160px" align="center" label="添加时间">
          <template slot-scope="scope">
            <span>{{ scope.row.create_time | parseTime('{y}-{m}-{d} {h}:{i}:{s}') }}</span>
          </template>
        </el-table-column>

        <el-table-column class-name="status-col" label="状态" width="80">
          <template slot-scope="{row}">
            <el-tag :type="row.status | statusFilter" size="mini">
              {{ (row.status == 1) ? '启用' : '禁用'}}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column align="center" label="操作" width="250">
          <template slot-scope="scope">
            <el-button type="primary" size="mini" @click="handleDetail(scope.$index, scope.row)">
              详情
            </el-button>

            <el-button type="warning" size="mini" @click="handleDownload(scope.row.id)">
              下载
            </el-button>

            <el-button type="danger" size="mini" icon="el-icon-delete" style="margin-left:10px;" @click="handleDelete(scope.$index, scope.row)">
              删除
            </el-button>                
          </template>
        </el-table-column>
      </el-table>

      <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />
    </el-card>

    <el-dialog title="附件详情" :visible.sync="detail.dialogVisible">
      <detail :data="detail.data" />
    </el-dialog>    
  </div>
</template>

<script>
import waves from '@/directive/waves' // waves directive
import { parseTime, renderSize } from '@/utils'
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination
import Detail from '@/components/Larke/Detail'
import { 
  getAttachmentList, 
  getAttachmentDetail, 
  deleteAttachment, 
  enableAttachment,
  disableAttachment,
  getAttachmentDowncode,
  getAttachmentDownloadUrl
} from '@/api/attachment'

export default {
  name: 'AdminLogIndex',
  components: { Pagination, Detail },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        1: 'success',
        0: 'danger'
      }
      return statusMap[status]
    },  
  },
  data() {
    return {
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        searchword: '',
        order: 'ASC',
        status: '',
        page: 1,
        limit: 10
      },
      statusOptions: [
        { key: 'open', display_name: '启用' },
        { key: 'close', display_name: '禁用' },
      ],    
      sortOptions: [
        { label: '正序', key: 'ASC' }, 
        { label: '倒叙', key: 'DESC' }
      ],
      detail: {
        dialogVisible: false,
        data: [],
      },
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      getAttachmentList({
        searchword: this.listQuery.searchword,    
        status: this.listQuery.status,
        order: this.listQuery.order,
        start: (this.listQuery.page - 1) * this.listQuery.limit,
        limit: this.listQuery.limit
      }).then(response => {
        this.list = response.data.list
        this.total = response.data.total
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },  
    handleDetail(index, row) {
      getAttachmentDetail(row.id).then((res) => {
        this.detail.dialogVisible = true
        const data = res.data

        this.detail.data = [
          {
            name: 'ID',
            content: data.id,
            type: 'text',
          },          
          {
            name: '属于',
            content: data.belong_type + '：' + data.belong_id,
            type: 'text',
          },
          {
            name: '文件名',
            content: data.name,
            type: 'text',
          },    
          {
            name: '存储位置',
            content: data.path,
            type: 'text',
          }, 
          {
            name: '文件Mime',
            content: data.mime,
            type: 'text',
          },                                       
          {
            name: '文件类型',
            content: data.extension,
            type: 'text',
          },
          {
            name: '文件大小',
            content: data.size,
            type: 'size',
          },
          {
            name: '文件md5',
            content: data.md5,
            type: 'text',
          },
          {
            name: '文件sha1',
            content: data.sha1,
            type: 'text',
          },  
          {
            name: '存储驱动',
            content: data.driver,
            type: 'text',
          },                              
          {
            name: '附件URL',
            content: data.url,
            type: 'text',
          },      
          {
            name: '激活状态',
            content: data.status,
            type: 'boolen',
          }, 
          {
            name: '最后更新',
            content: data.update_time,
            type: 'time',
          },    
          {
            name: '更新IP',
            content: data.update_ip,
            type: 'text',
          },                             
          {
            name: '添加时间',
            content: data.create_time,
            type: 'time',
          },    
          {
            name: '添加IP',
            content: data.create_ip,
            type: 'text',
          },                                    
        ]
      })
    },
    handleDownload(id) {
      if (id == '') {
        this.$message({
          message: '请选择要下载的附件',
          type: 'error',
          duration: 3 * 1000,
        })
        return ;
      }

      getAttachmentDowncode(id).then((res) => {
        const code = res.data.code
        const url = getAttachmentDownloadUrl(code)

        window.open(url, '_blank')
      })
    },
    handleDelete(index, row) {
      const thiz = this
      this.$confirm('确认要删除该附件吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteAttachment(row.id).then(res => {
          this.$message({
            message: res.message,
            type: 'success',
            duration: 3 * 1000,
            onClose() {
              thiz.list.splice(index, 1)
            }
          })
        })
      }).catch(() => {

      })
    },      
  }
}
</script>

<style scoped>
.pagination-container {
  padding: 5px 2px;
}
.edit-input {
  padding-right: 100px;
}
.cancel-btn {
  position: absolute;
  right: 15px;
  top: 10px;
}
</style>
