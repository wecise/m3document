<template>
    <el-container style="height: calc(100vh - 80px);padding: 0px;">
        <el-main style="padding:0px;overflow: hidden;background-color:#f2f2f2;">
            <el-container style="height:100%;">
                <el-main style="overflow:hidden;padding-bottom:0px;padding-right:0px;">
                    <DocumentView :model="selectedDoc" :auth="auth" v-if="selectedDoc"></DocumentView>
                    <Welcome v-else></Welcome>
                </el-main>
                <el-aside style="width:300px;overflow:hidden;background:#f2f2f2;" ref="leftView">
                    <el-container style="height:100%;">
                        <el-main>
                            <DocumentTree @open-doc="onOpen" @init="onInit" :auth="auth"></DocumentTree>
                        </el-main>
                    </el-container>
                </el-aside>
            </el-container>
        </el-main>
    </el-container>    
</template>

<script>
import _ from 'lodash';

import DocumentTree from './DocumentTree';
import DocumentView from './DocumentView';
import Welcome from './Welcome';

export default {
    name: 'index',
    props:{
        auth: Object
    },
    components: {
        DocumentTree,
        DocumentView,
        Welcome
    },
    data(){
        return {
            selectedDoc: null
        }
    },
    methods: {
        onInit(data){
            this.model = data;
        },
        onOpen(data){
            this.selectedDoc = data;
        }
    }
}
</script>

<style scoped>

</style>

<style>
    .el-dialog__wrapper{
        overflow: hidden!important;;
        height: 80vh!important;
    }
    .document-view-dialog.el-dialog{
        height: 100%;    
    }
    .document-view-dialog.el-dialog > .el-dialog__body{
        height: calc(100% - 80px); 
    }
    .document-view-dialog.el-dialog > .el-dialog__body > .el-container{
        height: 100%; 
    }
</style>
