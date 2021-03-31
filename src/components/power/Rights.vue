<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home}' }">首頁</el-breadcrumb-item>
      <el-breadcrumb-item>權限管理</el-breadcrumb-item>
      <el-breadcrumb-item>權限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card shadow="always" :body-style="{ padding: '20px' }">
        <el-table :data="rightsList" border stripe>
            <el-table-column type="index">
            </el-table-column>
            <el-table-column label="權限名稱" prop="authName">
            </el-table-column>
            <el-table-column label="路徑" prop="path">
            </el-table-column>
            <el-table-column label="權限等級" prop="level">
                <template slot-scope="scope">
                    <el-tag v-if="scope.row.level === '0'" size="normal">
                        一級
                    </el-tag>
                    <el-tag v-else-if="scope.row.level === '1'" type="success" size="normal">
                        二級
                    </el-tag>
                    <el-tag v-else type="warning" size="normal">
                        三級
                    </el-tag>
                </template>
            </el-table-column>
        </el-table>
        
    </el-card>
    
  </div>
</template>

<script>
export default {
    data() {
        return {
            rightsList: []
        }
    },
    created() {
        this.getRightsList()
    },
    methods: {
        async getRightsList() {
            const {data: res} = await this.$http.get('rights/list')
            if (res.meta.status !== 200) return this.$message.error('獲取權限失敗')
            this.rightsList = res.data
        }
    },
}
</script>

<style></style>
