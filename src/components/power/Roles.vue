<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>

      <el-table :data="roleList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', i1===0?'bdtop':'', 'vcenter']"
                    v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <!--通过for嵌套渲染二级权限-->
                <el-row :class="[i2===0?'':'bdtop', 'vcenter']"
                        v-for="(item2, i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag closable @close="removeRightById(scope.row, item2.id)"
                            type="success">{{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable @close="removeRightById(scope.row, item3.id)" type="warning"
                            v-for="(item3, i3) in item2.children"
                            :key="item3.id">{{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting"
                       @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!--分配权限对话框-->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%">
      <!--树形控件-->
      <el-tree show-checkbox :data="rightsList" :props="treeProps" node-key="id"
               default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: 'Roles',
    data () {
      return {
        roleList: [],
        setRightDialogVisible: false,
        rightsList: [],
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        //默认选中的id值数组
        defKeys: [],
        //选中的角色的id
        roleId: ''
      }
    },
    created () {
      this.getRolesList()
    },
    methods: {
      async getRolesList () {
        const {data: res} = await this.$http.get('roles')
        if (res.meta.status !== 200) {
          this.$message.error('获取角色列表失败')
        } else {
          this.roleList = res.data
        }
      },
      //根据ID删除对应的权限
      async removeRightById (role, rightId) {
        const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(error => error)
        if (confirmResult !== 'confirm') {
          this.$message.info('取消删除')
        } else {
          const {data: res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
          if (res.meta.status !== 200) {
            this.$message.error('删除权限失败')
          } else {
            role.children = res.data
          }
        }
      },

      async showSetRightDialog (role) {
        this.roleId = role.id
        const {data: res} = await this.$http.get('rights/tree')
        if(res.meta.status !== 200){
          return this.$message.error('获取角色权限失败')
        }
        else{
          //把获取到的权限数据保存到data中
          this.rightsList = res.data
          this.defKeys = []
          this.getLeafKeys(role, this.defKeys)
          this.setRightDialogVisible = true
        }
      },
      //获取所有角色的三级权限的id
      getLeafKeys(node, arr){
        if(!node.children){
          return arr.push(node.id)
        }
        else{
          node.children.forEach(item => {
            this.getLeafKeys(item, arr)
          })
        }
      },
      //点击为角色分配权限
      async allotRights(){
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ]
        const idStr = keys.join(',')
        const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})
        if(res.meta.status !== 200){
          return this.$message.error('分配角色权限失败')
        }
        else {
          this.$message.success('分配角色权限成功')
          this.getRolesList()
          this.setRightDialogVisible = false
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  .el-tag {
    margin: 7px;
  }

  .bdtop {
    border-top: 1px solid #eeeeee;
  }

  .bdbottom {
    border-bottom: 1px solid #eeeeee;
  }

  .vcenter {
    display: flex;
    align-items: center;
  }
</style>
