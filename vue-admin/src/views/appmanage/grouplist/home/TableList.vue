<template>
<div class="project-table table-wrap">
    <div class="box-table">
        <el-table height="100%" :data="tableData.rows" tooltip-effect="dark" v-loading="tableData.loading" @sort-change="tableSortChange" :default-sort="tableSortCondition">
            <template slot="empty">
                <no-data :types="tableData.isSearch">
                    <span>暂无数据</span>
                </no-data>
            </template>
            <el-table-column prop="groupName" label="群名称" show-overflow-tooltip></el-table-column>
            <el-table-column prop="operUser" label="群主" show-overflow-tooltip></el-table-column>
            <el-table-column prop="lastOperUser" label="操作人" show-overflow-tooltip></el-table-column>
            <el-table-column prop="operTime" label="创建时间" min-width="130" show-overflow-tooltip sortable></el-table-column>
            <el-table-column prop="ownerUserName" label="描述" show-overflow-tooltip></el-table-column>
            <el-table-column label="操作" min-width="130">
                <template slot-scope="scope">
                    <el-button type="text" @click="handleToSet(scope)">群设置</el-button>
                </template>
            </el-table-column>
        </el-table>
    </div>
    <div class="box-pagenation" :class="{'box-pagenation-1': !menuMode}" v-if="tableData.rows.length>0">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="tableData.currentPage" :page-size="tableData.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="tableData.total">
        </el-pagination>
    </div>
</div>
</template>

<script>
import {
    mapGetters,
    mapMutations
} from 'vuex'
import NoData from '@/components/NoData'
export default {
    name: 'ProjectsTable',
    components: { NoData },
    props: {
        tableData: {
            type: Object,
            required: true
        }
    },
    computed: {
        ...mapGetters({
            menuMode: 'common/getMenuMode',
            tableSortCondition: 'common/getTableSortCondition'
        })
    },
    methods: {
        ...mapMutations({
            setTableSortCondition: 'common/setTableSortCondition'
        }),
        handleCurrentChange (val) {
            this.tableData.currentPage = val
            this.$emit('flash')
        },
        handleSizeChange (val) {
            this.tableData.pageSize = val
            this.$emit('flash')
        },
        handleToSet (scope) {
            this.$router.push(`/groupListSet/${scope.row.id}`)
            localStorage.setItem('groupInfo', JSON.stringify(scope.row))
        },
        tableSortChange (val) {
            let data = {
                prop: val.prop,
                order: val.order
            }
            this.setTableSortCondition(data)
        }
    }
}
</script>
