<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <!-- 添加角色按钮区 -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 角色列表区 -->
            <el-table :data="rolesList" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                            <!-- 渲染一级权限 -->
                            <el-col :span="5">
                                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级、三级权限 -->
                            <el-col :span="19">
                                <!-- 嵌套渲染二级权限 -->
                                <el-row :class="[ i2 === 0 ? '': 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id"  closable @close="removeRightById(scope.row, item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <!-- 编辑按钮 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditRoles(scope.row.id)">编辑</el-button>
                        <!-- 删除按钮 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoles(scope.row.id)">删除</el-button>
                        <!-- 分配权限按钮 -->
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 弹框区域 -->
            <!-- 添加角色弹框 -->
            <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%">
                <el-form :model="addForm" :rules="addFromRules" ref="addFormRef" label-width="70px">
                    <el-form-item label="角色名称" prop="roleName" label-width="80px">
                        <el-input v-model="addForm.roleName"></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述" prop="roleDesc" label-width="80px">
                        <el-input v-model="addForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="addDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addRoles">确 定</el-button>
                </span>
            </el-dialog>
            <!-- 编辑角色弹框 -->
            <el-dialog title="编辑角色" :visible.sync="editDialogVisible" width="50%">
                <el-form :model="editForm" :rules="editFromRules" ref="editFormRef" label-width="70px">
                    <el-form-item label="角色名称" prop="roleName" label-width="80px">
                        <el-input v-model="editForm.roleName"></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述" prop="roleDesc" label-width="80px">
                        <el-input v-model="editForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="editDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editRolesInfo">确 定</el-button>
                </span>
            </el-dialog>
            <!-- 分配权限的弹框 -->
            <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
                <!-- 树形控件 -->
                <el-tree
                :data="rightsList"
                :props="treeProps"
                show-checkbox node-key="id"
                default-expand-all
                :default-checked-keys="defKeys"
                ref="treeRef">
                </el-tree>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="setRightDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="allotRights">确 定</el-button>
                </span>
            </el-dialog>
        </el-card>
    </div>
</template>

<script>
export default {
    data () {
        return {
            // 所有角色列表数据
            rolesList: [],
            // 控制添加角色弹框显示与隐藏
            addDialogVisible: false,
            // 添加角色的表单数据
            addForm: {
                roleName: '',
                roleDesc: ''
            },
            // 添加角色的验证规则对象
            addFromRules: {
                roleName: [
                    { required: true, message: '请输入角色名称', trigger: 'blur' }
                ],
                roleDesc: [
                    { required: true, message: '请输入角色描述', trigger: 'blur' }
                ]
            },
            // 查询到的角色信息对象
            editForm: {},
            // 控制编辑角色弹框显示与隐藏
            editDialogVisible: false,
            // 编辑角色的验证规则对象
            editFromRules: {
                roleName: [
                    { required: true, message: '请输入角色名称', trigger: 'blur' }
                ],
                roleDesc: [
                    { required: true, message: '请输入角色描述', trigger: 'blur' }
                ]
            },
            // 控制分配权限弹框的显示与隐藏
            setRightDialogVisible: false,
            // 所有权限的数据
            rightsList: [],
            // 树形控件的绑定对象
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            // 默认选中的节点的ID值数值(三级权限的id)
            defKeys: [],
            // 当前即将分配权限的ID
            roleId: []
        }
    },
    created () {
        this.getRolesList()
    },
    methods: {
        // 获取角色列表
        async getRolesList() {
            const { data: res } = await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败！')
            }
            this.rolesList = res.data
        },
        // 点击确定添加角色
        addRoles() {
            this.$refs.addFormRef.validate(async valid => {
                if (!valid) return
                // 可以发起请求
                const { data: res } = await this.$http.post('roles', this.addForm)
                if (res.meta.status !== 201) {
                   return this.$message.error('添加角色失败！')
                }
                this.$message.success('添加角色成功！')
                // 关闭弹框
                this.addDialogVisible = false
                // 重新获取角色信息
                this.getRolesList()
            })
        },
        // 编辑按钮
        async showEditRoles(id) {
            const { data: res } = await this.$http.get('roles/' + id)
            if (res.meta.status !== 200) {
                this.$message.error('获取角色信息失败！')
            }
            this.editForm = res.data
            this.editDialogVisible = true
        },
        // 点击确定完成编辑
        async editRolesInfo() {
            this.$refs.editFormRef.validate(async valid => {
                if (!valid) return
                const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {
                    roleName: this.editForm.roleName,
                    roleDesc: this.editForm.roleDesc
                })
                if (res.meta.status !== 200) {
                   return this.$message.error('编辑角色失败！')
                }
                // 关闭弹框
                this.editDialogVisible = false
                // 重新获取角色数据
                this.getRolesList()
                this.$message.success('编辑角色成功！')
            })
        },
        // 根据ID删除角色
        async removeRoles(id) {
            // 弹框询问用户是否删除
            const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const { data: res } = await this.$http.delete('roles/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error('删除角色失败！')
            }
            this.$message.success('删除角色成功！')
            this.getRolesList()
        },
        // 根据ID删除对应的权限
        async removeRightById(role, rightId) {
            // 弹框提示用户是否要删除
            const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除！')
            }
            const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
            if (res.meta.status !== 200) {
                return this.$message.error('删除权限失败！')
            }
            // 重新加载权限信息
            role.children = res.data
        },
        // 展示分配权限的对话框
        async showSetRightDialog(role) {
            // 保存当前即将分配权限的ID，供后面使用
            this.roleId = role.id
            const { data: res } = await this.$http.get('rights/tree')
            if (res.meta.status !== 200) {
                return this.$message.error('获取权限失败！')
            }
            // 把获取到的权限数据保存到rightsList中
            this.rightsList = res.data
            // 递归获取三级节点
            this.getLeafKeys(role, this.defKeys)
            // 显示弹框
            this.setRightDialogVisible = true
        },
        // 通过递归的形式，获取角色下所有三级权限的ID，保存到defKeys数组中
        getLeafKeys(node, arr) {
            // 如果当前node节点不包括children属性，则是三级节点
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item =>
                this.getLeafKeys(item, arr))
        },
        // 监听分配权限对话框的关闭事件
        setRightDialogClosed() {
            this.defKeys = []
        },
        // 点击角色分配权限
        async allotRights() {
            const keys = [
                this.$refs.treeRef.getCheckedKeys(),
                this.$refs.treeRef.getHalfCheckedKeys()
            ]
            // 将所有数据装换为用逗号隔开的字符串
            const idStr = keys.join(',')
            const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
            if (res.meta.status !== 200) {
                return this.$message.error('分配权限失败！')
            }
            this.$message.success('分配权限成功！')
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}

.bdtop {
    border-top: 1px solid #eee;
}

.bdbottom {
    border-bottom: 1px solid #eee;
}

.vcenter {
    display: flex;
    align-items: center;
}
</style>