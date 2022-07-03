<!--<template>-->
<!--    <div>-->
<!--    <el-dialog :visible.sync="diaVisible" width="30%">-->
<!--        <el-table  width="30%"-->
<!--                   :data="sendData"-->
<!--        >-->
<!--            <el-table-column-->
<!--                prop = "id"-->
<!--                label="id"-->
<!--                width="100">-->
<!--                <template scope="scope">-->
<!--                    <span>{{scope.row.id}}</span >-->
<!--                </template>-->
<!--            </el-table-column>-->
<!--            <el-table-column-->
<!--                prop="group"-->
<!--                label="group"-->
<!--                width="100">-->
<!--                <template scope="scope">-->
<!--                    <span>{{scope.row.group}}</span >-->
<!--                </template>-->
<!--            </el-table-column>-->
<!--            <el-table-column-->
<!--                prop="data"-->
<!--                label="data"-->
<!--                width="100">-->
<!--                <template scope="scope">-->
<!--                    <span>{{scope.row.data}}</span >-->
<!--                </template>-->
<!--            </el-table-column>-->
<!--            <el-table-column-->
<!--                fixed="right"-->
<!--                label="操作"-->
<!--                width="100">-->
<!--                <template scope="scope">-->
<!--                    <el-button @click="portTableRowEdit(scope.row,scope.$index)" type="text" size="small">修改</el-button>-->
<!--                    <el-button @click="portTableRowDelete(scope.row,scope.$index)" type="text" size="small">删除</el-button>-->
<!--                </template>-->
<!--            </el-table-column>-->
<!--        </el-table>-->
<!--    </el-dialog>-->
<!--    &lt;!&ndash;弹窗数据&ndash;&gt;-->
<!--    <el-dialog title="修改" :visible.sync="dialogFormVisible">-->
<!--        <el-form :model="editObj">-->
<!--            <el-form-item label="id">-->
<!--                <el-input v-model="editObj.id" auto-complete="off"></el-input>-->
<!--            </el-form-item>-->
<!--            <el-form-item label="group">-->
<!--                <el-input v-model="editObj.group" auto-complete="off"></el-input>-->
<!--            </el-form-item>-->
<!--            <el-form-item label="data">-->
<!--                <el-input v-model="editObj.data" auto-complete="off"></el-input>-->
<!--            </el-form-item>-->
<!--        </el-form>-->
<!--        <div slot="footer" class="dialog-footer">-->
<!--            <el-button @click="dialogFormVisible = false">取 消</el-button>-->
<!--            <el-button type="primary" @click="portTableRowEditDo">确 定</el-button>-->
<!--        </div>-->
<!--    </el-dialog>-->
<!--    </div>-->
<!--</template>-->

<!--<script>-->
<!--export default {-->
<!--    name: "tableData.vue",-->
<!--    props:['sendData','sendDialogVisible'],-->
<!--    data(){-->
<!--        return {-->
<!--            diaVisible : false,-->
<!--            dialogFormVisible:false,-->
<!--            editObj:{-->
<!--                id:"",-->
<!--                group:"",-->
<!--                data:""-->
<!--            }-->
<!--        }-->
<!--    },-->
<!--    methods: {-->
<!--        portTableRowEdit(row, index) {-->
<!--            this.$emit('editTable',row,index)-->
<!--            //记录索引-->
<!--            this.listIndex=index;-->
<!--            //记录数据-->
<!--            this.editObj=row;-->
<!--            //显示弹窗-->
<!--            this.dialogFormVisible=true;-->
<!--        },-->
<!--        portTableRowEditDo(){-->
<!--            let index=this.listIndex-->
<!--            //根据索引，赋值到list制定的数-->
<!--            this.sendData[index]=this.editObj;-->
<!--            //关闭弹窗-->
<!--            this.dialogFormVisible=false;-->
<!--        },-->
<!--        openReport() {-->
<!--            this.diaVisible = true-->
<!--        },-->
<!--        portTableRowDelete(row) {-->
<!--            this.sendData.splice(row,1)-->
<!--        },-->
<!--    },-->
<!--}-->
<!--</script>-->

<!--<style scoped>-->

<!--</style>-->
