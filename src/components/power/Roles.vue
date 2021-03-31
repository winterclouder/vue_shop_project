<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home}' }">首頁</el-breadcrumb-item>
      <el-breadcrumb-item>權限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="12" :offset="0">
          <el-button type="primary" size="default" @click="addNewRole"
            >新增角色</el-button
          >
        </el-col>
        <el-col :span="12" :offset="0"></el-col>
      </el-row>
      <el-table :data="roleList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom','vcenter', i1 === 0 ? 'bdtop' : '']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <el-col :span="5">
                <el-tag 
                  closable
                  @close="removeRightByid(scope.row, item.id)"
                >
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  :class="[i2 !== 0 ? 'bdtop' : '','vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag 
                      type="success" 
                      closable
                      @close="removeRightByid(scope.row, item2.id)"
                    >
                      {{
                        item2.authName
                      }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-row
                      :class="[i3 !== 0 ? 'bdtop' : '']"
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                    >
                      <el-col>
                        <el-tag 
                          type="warning" 
                          closable
                          @close="removeRightByid(scope.row, item3.id)"
                        >
                          {{
                            item2.authName
                          }}
                        </el-tag>
                      </el-col>
                    </el-row>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"> </el-table-column>
        <el-table-column label="角色名稱" prop="roleName"> </el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"> </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="socpe">
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="addNewRole"
              >編輯</el-button
            >
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="deleteRole"
              >刪除</el-button
            >
            <el-button
              type="warning"
              size="mini"
              icon="el-icon-setting"
              @click="showSetRightDialog(socpe.row)"
              >分配權限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog
      v-if="setRightDialogVisible"
      :visible.sync="setRightDialogVisible"
      title="分配權限"
      width="50%"
      @close="setRightDialogClosed"
    >
      <el-tree  
        ref="treeRef"
        :data="rightsList" 
        :props="treeProps" 
        :default-checked-keys="defKeys"
        show-checkbox
        node-key="id"
        default-expand-all
      >
      </el-tree>
      
      <span slot="footer">
        <el-button @click="setRightDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="allotRights">確定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      setRightDialogVisible: false,
      roleList: [],
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      roleId: ''
    }
  },
  methods: {
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idStr = keys.join(',')
      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})
      if (res.meta.status !== 200) {
        return this.$message.error('分配權限失敗')
      }
      this.$message.success('分配權限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    setRightDialogClosed() {
      this.defKeys = []
    },
    getLeafKeys(node, arr) {
      if(!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item,arr))

      // this.defKeys = []
    },
    addNewRole() {},
    deleteRole(id) {

    },
    async removeRightByid(role, rightId) {
      const confirmResult = await this.$confirm('此操作將刪除改文件，是否繼續',
        '提示',
        {
          confirmButtonText: '確定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(e => e)
        if (confirmResult !== 'confirm') {
          return this.$message.info('取消刪除')
        }
      this.$message.success('確認刪除')
      const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      console.log(res)
      if(res.meta.status !== 200 ) {
        this.$message.error('刪除權限失敗')
      }
      role.children = res.data
    },
    async showSetRightDialog(role) {
      this.roleId = role.id
      const {data:res} = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('獲取失敗')
      }
      this.rightsList = res.data

      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('角色獲取失敗')
      this.roleList = res.data
    }
  },
  created() {
    this.getRolesList()
  }
}
</script>

<style>
.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.el-tag {
  margin: 7px;
}
.vcenter {
    display: flex;
    align-items: center;
}
</style>
