<template>
<div class="project-table table-wrap">
    <div class="box-table">
        <el-table height="100%" :data="tableData.rows" tooltip-effect="dark" v-loading="tableData.loading" @sort-change="tableSortChange" @selection-change="handleSelected" :default-sort="tableSortCondition">
            <template slot="empty">
                <no-data :types="tableData.isSearch">
                    <span>暂无数据</span>
                </no-data>
            </template>
            <el-table-column type="selection" width="55"></el-table-column>
            <el-table-column prop="userId" label="userId" show-overflow-tooltip></el-table-column>
            <el-table-column prop="userName" label="userName" show-overflow-tooltip></el-table-column>
            <el-table-column prop="nickName" label="nickName" show-overflow-tooltip></el-table-column>
            <el-table-column prop="money" label="money" show-overflow-tooltip></el-table-column>
            <el-table-column prop="imgUrl" label="imgUrl" show-overflow-tooltip></el-table-column>
            <el-table-column prop="isLogin" label="isLogin" show-overflow-tooltip></el-table-column>
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
        handleSelected (val) {
            this.$emit('select', val)
        },
        handleCurrentChange (val) {
            this.tableData.currentPage = val
            this.$emit('flash')
        },
        handleSizeChange (val) {
            this.tableData.pageSize = val
            this.$emit('flash')
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
