<template>
  <div class="hello"> 
    <el-row>
      <el-col :span="10">
        <el-input v-model="input" placeholder="请输入姓名内容"></el-input>
      </el-col>
      <el-col :span="2">
        <el-button type="primary" @click="search()">查询</el-button>
      </el-col> 
      <el-col :span="2">
        <el-button type="primary" @click="add">新增</el-button>
      </el-col> 
      <el-col :span="2">
        <el-button type="primary" @click="showAll">全部</el-button>
      </el-col>
    </el-row>
    <el-row :gutter="10">
      <el-col :span="16" style="margin-top:10px;">
        <el-table :data="tableData" border align="left" style="width: 100%">
          <el-table-column prop="time" label="日期" > </el-table-column>
          <el-table-column  prop="name"  label="姓名"  >  </el-table-column>
          <el-table-column prop="address" label="地址"> </el-table-column> 
          <el-table-column fixed="right" label="操作"  >
            <template slot-scope="scope">
              <el-button  type="text" size="small" v-if="scope.row.status == '0'" @click="able(scope)">启用</el-button>
              <el-button  type="text" size="small" style="color:red;" v-if="scope.row.status != '0'" @click="able(scope)">禁用</el-button>
              <el-button type="text" size="small" @click="edit(scope)">编辑</el-button>
              <el-button type="text" size="small" @click="del(scope)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-col> 
    </el-row>
    
    <el-row>
      <div>
        <el-col :span="16" style="margin-top:15px;">
          <el-pagination
              layout="prev, pager, next"
              :total="total" 
              @current-change="handleCurrentChange"
              :current-page.sync="pageIndex"
              :page-size="pageSize"
              >
            </el-pagination>
        </el-col>
      </div> 
    </el-row>

    <el-dialog
      title="新增、编辑"
      :visible.sync="dialogVisible"
      width="30%"
      @open="open"
      :before-close="handleClose">
      <el-form :model="formData" status-icon label-width="100px">
        <el-form-item label="时间" >
          <el-input v-model="formData.time" ></el-input>
        </el-form-item> 
        <el-form-item label="姓名" >
          <el-input v-model="formData.name" ></el-input>
        </el-form-item>
        <el-form-item label="地址" >
          <el-input v-model="formData.address" ></el-input>
        </el-form-item> 
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cancel()">取 消</el-button>
        <el-button type="primary" @click="ok()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default { 
  data () {
    return { 
      tableData: [],
      input: '',
      dialogVisible: false,
      formData: {
        time: '',
        name: '',
        address: '',
        id: '',
        status: '',
      }, 
      flag: '',
      copy: {},
      total: 0,
      pageIndex: 1,
      pageSize: 4,
    }
  },
  methods: {
    handleCurrentChange(index) {
      console.log(index)
      this.pageIndex = index
      this.getData()

    },
    search() {
      this.pageIndex = 1;
      this.getData(); 
    },
    handleClose() {
      this.dialogVisible = false
      this.formData = {}
      console.log('取消')
    },
    open() {
      if (this.flag == 'add') {
        this.formData = {}
      } else if (this.flag == 'edit') {
        this.formData = this.copy 
      }
      console.log(this.formData)
    },
    ok() { 
      if (!this.formData.time || !this.formData.name || !this.formData.address) {
        alert('内容不能为空')
        return 
      } 
      let params = {
        time : this.formData.time,
        name: this.formData.name,
        address: this.formData.address, 
        id: this.formData.id, 
        status: this.formData.status,
      }
      
      if (this.flag == 'add') {
        params.status = '0'
        this.$axios.get('/api'+'/index/index/add', {params:params}).then((res) => {
          if (res.data.code != '0') { 
            this.tips(res)
            return
          } else {
            this.tips(res)
            this.getData()
          } 
          this.dialogVisible = false 
        }) 
      } else if(this.flag == 'edit') {  
        this.$axios.get('/api'+'/index/index/edit', {params:params}).then((res) => {
          if (res.data.code == '0') {
            this.tips(res)
            this.getData()
          } else {
            this.tips(res)
          } 
          this.dialogVisible = false 
        })
      }
    },
    cancel() {
      this.handleClose()
    },
    add() {
      this.dialogVisible = true
      this.flag = 'add'
    },
    del(scope) {
      let params = {
        id: scope.row.id
      }
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$axios.get('/api'+'/index/index/del', {params:params}).then((res) => {
          if (res.data.code != '0') { 
            this.tips(res)
            return
          } else {
            this.tips(res)

            this.pageIndex = 1
            this.getData()
          } 
        })
      }).catch(() => {})
    },
    edit(scope) {
      this.dialogVisible = true
      this.flag = 'edit'
      this.copy = JSON.parse(JSON.stringify(scope.row)) 
    },
    able(scope) {
      let params = {
        status : scope.row.status == 1 ? '0' : '1',
        id : scope.row.id
      }
      this.$axios.get('/api'+'/index/index/able', {params:params}).then((res) => {
        if (res.data.code != '0') { 
          this.tips(res)
          return
        } else {
          this.tips(res)
          this.getData()
        }  
      })
    },
    getData() {
      let params = {
        pageIndex: this.pageIndex,
        pageSize: this.pageSize,
        name: this.input
      }
      this.$axios.get('/api'+'/index/index/getData', {params:params}).then((res) => {
        console.log(res.data.data.data)
        if (res.data.code == '0') {
          this.tableData = res.data.data.data;
          this.total = res.data.data.total;
        } else {  
          alert(res.data.msg)
        } 
      })
    },
    showAll() {
      this.input = ''
      this.getData();
    },
    tips(res) {
      this.$notify({
        title: '提示',
        message: this.$createElement('i', { style: 'color: teal'}, res.data.msg)
      });
    }
  },
  mounted() {  
     this.getData()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
