<template>
    <el-container style="height:100%;">
        <el-header style="height:40px;line-height:40px;padding:0px 10px;">
            <el-input v-model="filterText" 
                placeholder="搜索" size="mini"
                clearable></el-input>
        </el-header>
        <el-main style="padding:0px 10px; height: 100%;">
            <el-tree :data="treeData" 
                    :props="defaultProps" 
                    node-key="fullname"
                    :highlight-current="true"
                    :default-expand-all="true"
                    @node-click="onNodeClick"
                    :filter-node-method="onFilterNode"
                    :expand-on-click-node="false"
                    style="background:transparent;"
                    ref="tree">
                <span slot-scope="{ node, data }" style="width:100%;height:30px;line-height: 30px;"  @mouseenter="onMouseEnter(data)" @mouseleave="onMouseLeave(data)">
                    <span v-if="data.ftype==='dir'">
                        <i class="el-icon-folder" style="color:#FFC107;"></i>
                        <span> {{node.label | pickShortLabel}}{{data | pickCount}}</span>
                        <el-dropdown v-show="data.show && m3.auth.isAdmin" style="float:right;width:14px;margin:0 5px;" trigger="click">
                            <span class="el-dropdown-link">
                                <i class="el-icon-more el-icon--right" style="color:#000000;"></i>
                            </span>
                            <el-dropdown-menu slot="dropdown">
                                <el-dropdown-item @click.native="onDelete(data,$event)" icon="el-icon-delete">删除</el-dropdown-item>
                                <el-dropdown-item @click.native="onNodeClick(data)" icon="el-icon-refresh">刷新</el-dropdown-item>
                                <el-dropdown-item @click.native="onNewFile(data,$event)" key="" icon="el-icon-plus">新建文件</el-dropdown-item>
                                <el-dropdown-item @click.native="onNewDir(data,$event)" icon="el-icon-folder-add">新建目录</el-dropdown-item>
                                <el-dropdown-item @click.native="onUpload(data,$event)" icon="el-icon-upload">上传</el-dropdown-item>
                            </el-dropdown-menu>
                        </el-dropdown>
                    </span>
                    <span v-else>
                        <i class="el-icon-c-scale-to-original" style="color:#0088cc;"></i>
                        <span> {{node.label | pickShortLabel}}</span>
                        <el-button v-show="data.show" type="text" @click.stop="onDownload(data,$event)" style="float:right;width:14px;margin:0 5px;" icon="el-icon-download"></el-button>
                        <el-button v-show="data.show && m3.auth.isAdmin" type="text" @click.stop="onDelete(data,$event)" style="float:right;width:14px;margin:0 5px;" icon="el-icon-delete"></el-button>
                    </span>
                </span> 
            </el-tree>
        </el-main>
    </el-container>
</template>

<script>
import _ from 'lodash';

export default {
    name: 'DocumentTree',
    props:{
        auth: Object
    },
    data() {
        return {
            defaultProps: {
                children: 'children',
                label: 'name'
            },
            treeData: [],
            root: "/assets/documents",
            filterText: "",
            dialog: {
                upload: {
                    show: false,
                    url: '',
                    fileList: [],
                    ifIndex: {index:true}
                } 
            }
        }
    },
    filters: {
        pickShortLabel(item){
            return _.truncate(item, {
                        'length': 16,
                        'omission': ' ...'
                    });
        },
        pickCount(data){
            try{
                let count = data.children.length;
                return count>0?`(${data.children.length})`:'';
            } catch(err){
                return '';
            }
        }
    },
    watch: {
        filterText(val) {
            if(_.isEmpty(val)){
                this.onInit();
            } else {
                this.$refs.tree.filter(val);
            }
        }
    },
    created(){
        this.onInit();
    },
    methods: {
        onMouseEnter(item){
            this.$set(item, 'show', true)
        },
        onMouseLeave(item){
            this.$set(item, 'show', false)
        },
        onRefresh(item,index){
            if(_.isEmpty(index)){
                this.initData();
            } else {
                this.m3.dfs.list({fullname:item.fullname}).then(rtn=>{
                    this.$set(item, 'children', rtn.message);
                });
            }
        },
        onNewDir(item,index){
            this.$prompt('请输入目录名称', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消'
                }).then(({ value }) => {

                if(_.isEmpty(value)){
                    this.$message({
                        type: 'warning',
                        message: '请输入目录名称！'
                    });
                    return false;
                }

                let ftype = 'dir';
                let attr = {remark: '', rate: '0'};

                let param = {
                    parent: item.fullname, name: value, 
                    data: { content: '', ftype: ftype, attr: attr, index: true }    
                };

                this.m3.dfs.write(param).then((rtn)=>{
                    this.$message({
                        type: "success",
                        message: "新建目录成功！"
                    })

                    // 刷新
                    this.onRefresh(item,index);
                        
                }).catch(err=>{
                    this.$message({
                        type: "error",
                        message: "新建目录失败，" + err
                    })
                });
                
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '取消新建目录操作'
                });       
            });
            
        },
        onNewFile(item,index){
            
            this.$prompt('请输入知识文件名称', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消'
                }).then(({ value }) => {

                if(_.isEmpty(value)){
                    this.$message({
                        type: 'warning',
                        message: '请输入名称！'
                    });
                    return false;
                }

               
                    
                let ftype = 'md';
                let attr = {remark: '', rate: '0'};
                let name = [value,ftype].join(".");

                let param = {
                    parent: item.fullname, name: name, 
                    data: { content: " ", ftype: ftype, attr: attr, index: true }    
                };

                this.m3.dfs.write(param).then(res=>{
                    
                    this.$message({
                        type: "success",
                        message: "新建成功！"
                    })

                    // 刷新
                    this.onRefresh(item,index);

                }).catch(err=>{
                    this.$message({
                        type: "error",
                        message: "新建失败，" + err
                    })
                })

                
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '取消新建文件操作'
                });       
            });
        },
        onDelete(item,index){
            this.$confirm(`确认要删除该知识：${item.name}？`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                
                this.m3.dfsDelete({parent:item.parent,name:item.name}).then(rtn=>{
                    
                    this.$message({
                        type: "success",
                        message: "删除成功！"
                    })
                    
                    // 刷新
                    try{
                        this.m3.dfs.list({fullname:item.parent}).then(res=>{
                            let childrenData = res.message;
                            let parent = this.$refs.tree.getNode(item.parent)
                            this.$set(parent.data, 'children', childrenData);
                        })
                    } catch(err){
                        this.initData();
                    }
                        
                }).catch(err=>{
                    this.$message({
                        type: "error",
                        message: "删除失败，" + err
                    })
                })
                

            }).catch((err) => {
                console.log(err)
            });
        },
        onUpload(item,index){
            /* const self = this;

            let wnd = null;
            let wndID = `jsPanel-upload-${objectHash.sha1(item.id)}`;

            try{
                if(jsPanel.activePanels.getPanel(wndID)){
                    jsPanel.activePanels.getPanel(wndID).close();
                }
            } catch(error){

            }
            finally{
                wnd = maxWindow.winUpload('文件上传', `<div id="${wndID}"></div>`, null, null);
            }
            
            new Vue({
                delimiters: ['#{', '}#'],
                template:   `<el-container>
                                <el-main>
                                    <el-upload drag
                                        multiple
                                        show-file-list="false"
                                        :action="upload.url"
                                        :data="upload.ifIndex"
                                        :on-success="onSuccess"
                                        :on-error="onError"
                                        :before-upload="onBeforeUpload"
                                        :on-remove="onRemove"
                                        list-type="text"
                                        name="uploadfile">
                                        <i class="el-icon-upload"></i>
                                    </el-upload>
                                </el-main>
                                <el-footer>
                                    <i class="fas fa-clock"></i> 上传文件：#{upload.fileList.length}# 
                                </el-footer>
                            </el-container>`,
                data: {
                    upload: {
                        url: `/fs/${item.fullname}?issys=true`,
                        fileList: [],
                        ifIndex: {index:true}
                    }
                },
                created(){
                    
                },
                methods: {
                    onBeforeUpload(file){
                        
                    },
                    onSuccess(res,file,FileList){
                        this.upload.fileList = FileList;
                        
                        _.forEach(FileList,(v)=>{
                            try{
                                
                                let attr = {remark: '', rate:'0'};
                                fsHandler.fsUpdateAttr(item.parent, v.name, attr);

                            } catch(err){
                                let attr = {remark: '', rate:'0'};
                                fsHandler.fsUpdateAttr(item.parent, v.name, attr);
                            }
                        })

                        // 刷新
                        self.onRefresh(item,index);

                    },
                    onError(res,file,FileList){
                        

                    },
                    onRemove(file, fileList) {
                        fsHandler.fsDeleteAsync(item.fullname,file.name).then( (rtn)=>{
                            if(rtn == 1){
                                // 刷新
                                self.onRefresh(item,index);
                            }
                        } );
                        
                    },
                    onPreview(file) {
                        console.log(file);
                    }
                }
            }).$mount(`#${wndID}`);
             */
        },
        onDownload(item,index){
            let url = `/fs/${item.fullname}?type=download&issys=true`;
            window.open(url,"_blank");
        },
        onFilterNode:_.debounce(function(value, data) {
            if (!value) return true;

            try{
                this.m3.callFS("/matrix/m3document/getFsByTerm.js", encodeURIComponent(value)).then( rtn=>{
                    this.treeData = rtn.message;
                });
                
            } catch(err){
                this.treeData = [];
            }

        },1000),
        onNodeClick(data){
            if(!data.isdir) {
                this.$emit("open-doc", data);
            } else {
                this.m3.dfs.list({fullname:data.fullname}).then( rtn=>{
                    this.$refs.tree.updateKeyChildren(data.fullname,rtn.message);
                } );
            }
        },
        onInit(){
            this.m3.callFS("/matrix/m3document/getFsForTree.js", encodeURIComponent(this.root)).then( rtn=>{
                this.treeData = rtn.message; 
            });
        }
    }
}
</script>

<style scoped>

</style>