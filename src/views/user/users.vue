<template>
    <div>
        <!--面包屑-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>   
        </el-breadcrumb>

        <!--卡片视图-->
        <el-card>
            <!--搜索框-->

            <el-row :gutter="20">
                <el-col :span="10">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable
                    @clear="getUserList">
                        <template #prepend>
                            <el-select v-model="select" placeholder="请选择">
                                <el-option label="企业" value="1"></el-option>
                                <el-option label="个人" value="2"></el-option>
                            </el-select>
                        </template>
                        <template #append>
                            <el-button icon="el-icon-search" @click="getUserList"></el-button>
                        </template>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>

            <el-table :data="UserList" border stripe>
                <el-table-column type="index" label="序号"></el-table-column>
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态">
                    <template v-slot="scope">
                        <el-switch
                            v-model="scope.row.mg_state"
                            @change="userStateChanged(scope.row)">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" prop="" width="180px">
                    <template v-slot="scope">
                        <!--修改按钮-->
                        <el-button type="primary" icon="el-icon-edit" size="mini"
                        @click="showEditDialog(scope.row.id)"></el-button>
                        <!--删除按钮-->
                        <el-button type="danger" icon="el-icon-delete" size="mini"
                        @click="removeUserById(scope.row.id)"></el-button>
                        <!--分配角色-->
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
                        </el-tooltip>   
                    </template>
                </el-table-column>
            </el-table>

            <!--分页-->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>

        </el-card>

        <!--添加用户信息-->
        <el-dialog
            title="添加用户"
            v-model="addDialogVisible"
            @close="addDialogClosed"
            width="50%">
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px" >
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>    
            
            <!--底部区域-->
            <template #footer>
                <span class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
                </span>
            </template>
        </el-dialog>

        <!--修改用户信息-->
        <el-dialog
            title="修改用户"
            v-model="editDialogVisible"
            @close="editDialogClosed"
            width="50%">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>    

            <template #footer>
                <span class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUserInfo">确 定</el-button>
                </span>
            </template>
        </el-dialog>


    </div>
</template>

<script>
export default {
    data() {
        //验证邮箱
        var checkEmail = (rule, value, cb) => {
            // const regEmail = /^([a-zA-Z0-9_-]) + @([a-zA-Z0-9_-]) + (\.[a-zA-Z0-9_-])+/

            const regEmail = /^[a-zA-Z0-9_]{1,}[@][a-z0-9]{2,3}[.][a-z]{2,3}/
            if(regEmail.test(value)) {
                return cb()
            }
            cb(new Error('请输入合法邮箱'))
        }
        //验证手机号
        var checkMobile = (rule, value, cb) => {
            const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/

            if(regMobile.test(value)) {
                return cb()
            }
            cb(new Error('请输入合法手机号'))
        }

        return {
            //获取用户列表的参数对象
            queryInfo: {
                query: '',
                //当前页号
                pagenum: 1,
                pagesize: 2
            },

            UserList: [],
            total: 0,

            //控制添加用户对话框的显示与隐藏
            addDialogVisible: false,
            //添加用户的表单数据
            addForm: {
                username: '',
                password: '',
                email: '',
                mobile: ''
            },
            //添加表单的验证规则
            addFormRules: {
                username: [
                    {required: true, messgae: '请输入用户名', trigger: 'blur'},
                    {min: 3, max: 10, messgae: '用户名的长度在3-10个字符之间', trigger: 'blur'}
                ],
                password: [
                    {required: true, messgae: '请输入密码', trigger: 'blur'},
                    {min: 6, max: 10, messgae: '密码的长度在6-10个字符之间', trigger: 'blur'}
                ],
                email: [
                    {required: true, messgae: '请输入邮箱', trigger: 'blur'},
                    {validator: checkEmail, trigger: 'blur'}
                ],
                mobile: [
                    {required: true, messgae: '请输入手机号码', trigger: 'blur'},
                    {validator: checkMobile, trigger: 'blur'}
                ]
            },
            //监听修改用户对话框的显示
            editDialogVisible: false,
            //查询到的用户信息
            editForm: {},
            //修改表单验证规则
            editFormRules: {
                email: [
                    {require: true, messgae: '请输入邮箱', trigger: 'blur'},
                    {validator: checkEmail, trigger: 'blur'}
                ],
                mobile: [
                    {require: true, messgae: '请输入手机号', trigger: 'blur'},
                    {validator: checkMobile, trigger: 'blur'}
                ]
            }

        }
    },
    created() {
        this.getUserList()
    },
    methods : {
        async getUserList() {
            const {data : res} = await this.$http.get('users', {
                params: this.queryInfo
            })
            if (res.meta.status != 200) {
                return this.$message.error("获取用户列表失败")
            }
            this.UserList = res.data.users
            this.total = res.data.total
            // console.log(res)
        },
        //监听pagesize改变
        handleSizeChange(newSize) {
            // console.log(newSize)
            this.queryInfo.pagesize = newSize
            this.getUserList()
        },
        handleCurrentChange(newPage) {
            // console.log(newPage)
            this.queryInfo.pagenum = newPage
            this.getUserList()
        },
        //监听switch开关
        async userStateChanged(userInfo) {
            // console.log(userInfo)
            const {data : res} = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
            if (res.meta.status !== 200) {
                userInfo.mg_state = !userInfo.mg_state
                return this.$message.error("更新用户状态失败")
            }
            this.$message.success("更新用户状态成功")
        },
        //监听对话框的关闭（重置）
        addDialogClosed() {
            this.$refs.addFormRef.resetFields();
        },
        //添加新用户
        addUser() {
            this.$refs.addFormRef.validate(async valid =>{
                if (!valid) 
                    return
                const{data : res} = await this.$http.post('users', this.addForm) 
                if (res.meta.status !== 201) {
                    this.$messgae.error("添加用户失败")
                }   
                this.$message.success("添加用户成功")
                this.addDialogVisible = false
                //重新获取用户列表
                this.getUserList()
            })
        },
        //展示编辑用户的对话框
        async showEditDialog(id) {
            // console.log(id)
            const {data : res} = await this.$http.get('users/' + id)

            if (res.meta.status !== 200) {
                return this.$message.error("查询用户信息失败")
            }

            this.editForm = res.data
            this.editDialogVisible = true
        },
        editDialogClosed() {
             this.$refs.editFormRef.resetFields();
        },
        //修改用户信息
        editUserInfo() {
            this.$refs.editFormRef.validate(async valid => {
                if (!valid)
                    return 
                const {data : res} = await this.$http.put('users/' + this.editForm.id, {
                    email: this.editForm.email,
                    mobile: this.editForm.mobile
                })

                if(res.meta.status !== 200) {
                    return this.$message.error('更新用户数据失败')
                }
                this.editDialogVisible = false,
                this.getUserList(),
                this.$message.success('更新用户信息成功')
            })
        },
        //根据id删除用户信息
        async removeUserById(id) {
            //弹窗提示
            const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)

            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }

            const {data : res} = await this.$http.delete('users/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error("删除用户失败")
            }
            this.$message.success('删除用户成功')
            this.getUserList()
        }
    }
}
</script>

<style>

</style>