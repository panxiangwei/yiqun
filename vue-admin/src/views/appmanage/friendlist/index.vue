<template>
<div class="projects box-wrap">
    <el-form @submit.native.prevent>
        <el-form-item>
            <el-input
                @keyup.enter.native="handleSearch(true)"
                @clear="handleSearch(true)"
                placeholder="请输入关键字"
                clearable
                v-model="queryForm.name">
                <i @click="handleSearch(true)" slot="prefix" class="iconfont icon-sousuo"></i>
            </el-input>
        </el-form-item>
        <el-form-item>
            <el-button v-hasPermission="'friend:add'" type="primary" @click="resetForm(false)">加友</el-button>
            <el-button v-hasPermission="'friend:delete'" @click="handleDel">删除</el-button>
        </el-form-item>
    </el-form>
    <table-list :tableData="tableData" @flash="handleSearch" @select="handleSelect" @update="resetForm"></table-list>
    <add-dialog v-if="addForm.show" :addForm="addForm" @save="handleSave($event)"></add-dialog>
</div>
</template>

<script>
import { mapGetters } from 'vuex'
import AddDialog from './AddDialog'
import TableList from './TableList'
export default {
    name: 'ProjectsIndex',
    components: {
        AddDialog,
        TableList
    },
    data () {
        const role = localStorage.getItem('ROLES')
        return {
            role,
            tableData: {
                loading: false,
                total: 0,
                pageSize: 10,
                currentPage: 1,
                isSearch: false,
                rows: []
            },
            addForm: {
                show: false,
                title: '新增好友',
                form: {
                    userId: '',
                    friendId: ''
                }
            },
            queryForm: {
                name: ''
            },
            sortOrder: true,
            select: []
        }
    },
    computed: {
        ...mapGetters({
            menuMode: 'common/getMenuMode'
        })
    },
    created () {
        this.init()
    },
    methods: {
        handleSelect (val) {
            this.select = val.map(v => v.id)
        },
        handleDel () {
            this.$confirm('确定要删除该版本吗？', '提示', {
                confirmButtonText: '确 定',
                cancelButtonText: '取 消',
                type: 'warning'
            }).then(() => {
                this.confirmDel()
            }).catch(() => {})
        },
        confirmDel () {
            let params = {
                ids: this.select
            }
            this.$post('/friend/delete', params).then(res => {
                if (res) {
                    this.$message.success('删除成功')
                    this.addForm.show = false
                    this.init()
                }
            })
        },
        handleSave (friendIds) {
            let url = this.addForm.title === '新增' ? '/friend/add' : '/friend/update'
            this.$post(url, { friendIds: friendIds }).then(res => {
                if (res || this.addForm.title === '修改') {
                    this.$message.success(this.addForm.title === '新增' ? '新增成功' : '修改成功')
                    this.addForm.show = false
                    this.init()
                }
            })
        },
        handleSearch (flag) {
            this.tableData.isSearch = true
            this.init()
        },
        init () {
            const {
                name
            } = this.queryForm
            const {
                pageSize,
                currentPage
            } = this.tableData
            const params = {
                pageSize: pageSize,
                pageNum: currentPage,
                keyword: name
            }
            this.tableData.loading = true
            this.$post('/friend/query', params).then(res => {
                if (res) {
                    this.tableData.rows = res.rows
                    this.tableData.total = res.total
                }
            }).finally(() => {
                this.tableData.loading = false
            })
        },
        resetForm (item) {
            if (item) {
                this.addForm = {
                    show: true,
                    title: '修改',
                    form: item
                }
            } else {
                this.addForm = {
                    show: true,
                    title: '新增',
                    form: {
                        userId: '',
                        friendId: ''
                    }
                }
            }
        }
    }
}
</script>

<style lang="less">
.projects {
    .is-active {
        color: #2680EB !important;
    }
    .is-round {
        line-height: 22px;
        margin-right: 33px;
        padding: 0 12px;
        min-width: 0;
    }
    .el-form {
        display: flex;
        justify-content: space-between;
        .el-input {
            margin-right: 16px;
        }
        .el-button {
            margin-left: 33px;
        }
        .iconfont {
            cursor: pointer;
        }
        .icon-liebiao {
            margin-right: 12px;
        }
    }
}
</style>
