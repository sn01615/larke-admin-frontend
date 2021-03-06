<template>
  <el-form :model="data" :rules="rules" label-width="100px" ref="authRuleForm">
    <el-form-item label="父级权限" prop="parentid">
      <el-select v-model="data.parentid" placeholder="请选择" 
        clearable 
        @change="parentidChange"
        filterable 
        :filter-method="parentFilter">
        <el-option v-for="item in parentOptions" :key="item.key" :label="item.display_name | entityToString" :value="item.key" />
      </el-select>            
    </el-form-item>
    <el-form-item label="名称" prop="title">
      <el-input v-model.trim="data.title" placeholder="请填写权限名称" />
    </el-form-item>   
    <el-form-item label="请求链接" prop="url">
      <el-tooltip effect="dark" content="请求链接默认不用加前缀" placement="top">
        <el-input v-model.trim="data.url" placeholder="请填写请求链接" >
          <template slot="prepend">
            <i class="el-icon-link" />
          </template>
        </el-input>
      </el-tooltip>
    </el-form-item>                 
    <el-form-item label="请求方式" prop="method">
      <el-select v-model="data.method">
        <el-option v-for="item in methodOptions" :key="item.key" :label="item.display_name" :value="item.key" />
      </el-select>          
    </el-form-item>
    <el-form-item label="标识" prop="slug">
      <el-input v-model.trim="data.slug" placeholder="请填写标识" />
    </el-form-item>        
    <el-form-item label="权限描述" prop="description">
      <el-input type="textarea" v-model.trim="data.description" rows="6" placeholder="请填写权限描述"></el-input>
    </el-form-item>      
    <el-form-item label="排序" prop="listorder">
      <el-input v-model.trim="data.listorder" placeholder="请填写排序" />
    </el-form-item>      
    <el-form-item label="鉴定权限" prop="is_need_auth"> 
      <el-radio-group v-model="data.is_need_auth">
        <el-radio :label="1">启用</el-radio>
        <el-radio :label="0">否</el-radio>
      </el-radio-group> 
    </el-form-item>                      
    <el-form-item label="状态" prop="status"> 
      <el-radio-group v-model="data.status">
        <el-radio :label="1">启用</el-radio>
        <el-radio :label="0">禁用</el-radio>
      </el-radio-group>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submit">提交</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
import { 
  getRuleDetail,
  getRuleChildrenList,
  updateRule 
} from '@/api/authRule'

export default {
  name: 'AuthRuleEdit',
  components: { },
  props: {
    item: {
      type: Object,
      default: () => {
        return {}
      }
    }
  },   
  data() {   
    return {
      all: [],
      chilren: [],      
      id: '',
      parentid: '',
      data: {
        parentid: '',
        title: '',
        url: '',
        method: 'GET',
        slug: '',
        description: '',
        listorder: 100,
        is_need_auth: 1,
        status: 1,
      },
      rules:{
          parentid:[
            {required:true, message:'父级不能为空', trigger:'change'}
          ],
          title:[
            {required:true, message:'名称不能为空', trigger:'blur'}
          ],          
          url:[
            {required:true, message:'请求链接不能为空', trigger:'blur'}
          ],   
          slug:[
            {required:true, message:'标识不能为空', trigger:'blur'}
          ],    
          listorder:[
            {required:true, message:'排序不能为空', trigger:'blur'}
          ],                           
      },      
      parentOptions: [
        { key: '0', display_name: '顶级权限' },
      ],
      parentFilterOptions: [],
      methodOptions: [
        { key: 'GET', display_name: 'GET' },
        { key: 'HEAD', display_name: 'HEAD' },
        { key: 'POST', display_name: 'POST' },
        { key: 'PUT', display_name: 'PUT' },
        { key: 'DELETE', display_name: 'DELETE' },
        { key: 'PATCH', display_name: 'PATCH' },
        { key: 'OPTIONS', display_name: 'OPTIONS' },
      ],      
    }
  },
  watch: {
    item: {
      handler(val, oldVal) {
        if (this.item.dialogVisible == true) {
          this.id = val.id
          this.fetchParents().then(() => {
            this.fetchData(val.id)
          }) 
        }
      },
      deep: true
    }
  },
  created() {
    const id = this.item.id
    this.id = id
    this.initData().then(() => {
      this.fetchData(id)
    }) 
  },
  methods: {  
    initData() {
      return new Promise((resolve, reject) => {
        const all = this.getAll()
        const children = this.getChildren()   
        
        Promise.all([all, children])
          .then(([all, children]) => {
            this.all = all.list
            this.children = children.list

            this.fetchParents().then(() => {})

            resolve()
          })
          .catch(() => {})
      })
    },
    getAll() {
      return new Promise((resolve, reject) => {
        getRuleChildrenList({
          id: 0,
          type: 'list',
        }).then(res => {
          resolve(res.data)
        }).catch(err => {
          reject(err)
        })
      })  
    },
    getChildren() {
      return new Promise((resolve, reject) => {
        getRuleChildrenList({
          id: this.id,
          type: 'ids'
        }).then(res => {
          resolve(res.data)
        }).catch(err => {
          reject(err)
        })
      })     
    },    
    fetchParents() {
      return new Promise((resolve, reject) => {
        this.getChildren().then((res) => {
          this.children = res.list
  
          const all = this.all
          const children = this.children

          this.parentOptions = [
            { key: '0', display_name: '顶级权限' },
          ]
          this.parentFilterOptions = []

          children.push(this.id)    
          all.forEach(item => {
            if (! children.includes(item.id)) {
              this.parentOptions.push({
                key: item.id, 
                display_name: item.spacer + ' ' + item.title + '【' + item.method + '】'
              })     
            }      
          }); 

          this.parentFilterOptions = this.parentOptions 

          resolve()
        }) 
      })  
    },    
    fetchData(id) {
      getRuleDetail(id).then(response => {
        this.parentid = response.data.parentid
        this.data = response.data
      }).catch(err => {
        console.log(err)
      })
    },
    parentFilter(val) {
      this.data.parentid = val
      if (val) {
        this.parentOptions = this.parentFilterOptions.filter(item => {
          if (!!~item.display_name.indexOf(val)
            || !!~item.display_name.toUpperCase().indexOf(val.toUpperCase())
          ) {
            return true
          }
        })
      } else {
        this.parentOptions = this.parentFilterOptions
      }
    },
    parentidChange(val) {
      if (! val) {
        this.parentOptions = this.parentFilterOptions
        this.data.parentid = this.parentid
      }
    },    
    submit() {
      const thiz = this
      updateRule(this.id, {     
        parentid: this.data.parentid,
        title: this.data.title,
        url: this.data.url,
        method: this.data.method,
        slug: this.data.slug,
        description: this.data.description,
        listorder: this.data.listorder,
        is_need_auth: this.data.is_need_auth,
        status: this.data.status,
      }).then(response => {
        this.$message({
          message: '更新权限信息成功',
          type: 'success',
          duration: 5 * 1000,
          onClose() {
            if (thiz.$refs.authRuleForm !== undefined) {
              thiz.$refs.authRuleForm.resetFields()
            }            
            thiz.item.dialogVisible = false
          }
        })
      })
    }
  }
}
</script>
