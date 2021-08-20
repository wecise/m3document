<template>
    <el-container v-if="ready">
        <el-main style="padding: 0px;">
            <mavonEditor v-model="doc.content" 
                :defaultOpen="preview"
                style="height: 100%" 
                @save="onSave" 
                v-if="doc.content"></mavonEditor>
        </el-main>
        <el-footer style="height:30px;line-height:30px;">
            <span>
                <i class="el-icon-user"></i> {{doc.author}}
            </span>
            <el-divider direction="vertical"></el-divider>
            编辑于 {{ doc | pickTime }}
        </el-footer>
    </el-container>
</template>

<script>
import _ from 'lodash';
import { mavonEditor } from 'mavon-editor';
import 'mavon-editor/dist/css/index.css';

export default {
    name: 'DocumentView',
    props: {
        model: Object,
        auth: Object
    },
    data(){
        return {
            ready: false,
            doc: null
        }
    },
    components: {
        mavonEditor
    },
    filters:{
        pickTime(item){
            try{
                let mtime = item.mtime;
                return window.moment(mtime).format(window.global.register.format);
            } catch(err){
                return window.moment(item.vtime).format(window.global.register.format);
            }
        }
    },
    watch:{
        model: {
            handler(val){
                this.doc = val;
                this.m3.dfsRead({parent: this.doc.parent, name: this.doc.name}).then(res=>{
                    setTimeout(()=>{
                        this.ready = false;
                        this.ready = true;
                    },300)
                    return _.extend(this.doc, {content:res});
                })
            },
            immediate: true
        }
    },
    methods: {
        onSave(value,renderValue){
            try{
                let attr = null;
                
                if(_.isEmpty(this.doc.attr)){
                    attr = {remark: '', rate:1};
                } else {
                    attr = _.attempt(JSON.parse.bind(null, this.doc.attr));
                    if(attr.rate){
                        _.extend(attr, {rate:attr.rate + 1});    
                    } else {
                        _.extend(attr, {rate:1}); 
                    }
                }
                
                let param = {
                    parent: this.doc.parent, name: this.doc.name, 
                    data: { content: value, ftype: this.doc.ftype, attr: attr, index: true }    
                };

                this.m3.dfsWrite(param).then( rtn=>{
                    this.$message.success("提交成功");
                }).catch(err=>{
                    this.$message.error("提交失败，请确认。");
                });
                
            } catch(err){
                console.error(err);
            }
            
        }
    }

}
</script>
<style scoped>
    .el-container{
        height: 100%;
    }
</style>
<style>
    
    .document-view-dialog.el-dialog{
        width: 80%;
    }
</style>