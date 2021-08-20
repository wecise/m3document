<template>
    <el-container>
        <el-main align="center" class="container">
            
            <h2> 欢迎使用 M³ 在线文档创作</h2>

            <p>
                                        
                <el-button style="width:100px;height:90px;">
                    <i class="el-icon-money" style="font-size:48px;"></i> <p>文档组织</p>
                </el-button>

                <el-button style="width:100px;height:90px;">
                    <i class="el-icon-edit-outline" style="font-size:48px;"></i> <p>文档编辑</p>
                </el-button>
                
                <el-button style="width:100px;height:90px;">
                    <i class="el-icon-share" style="font-size:48px;"></i> <p>文档分享</p>
                </el-button>
            
            </p>

            <p>
                <el-image :src="require('@/assets/svg/bg.svg')">
                    <div slot="placeholder" class="image-slot">
                        加载中<span class="dot">...</span>
                    </div>
                </el-image>
            </p>

            <p>
                如有任何意见或建议，请及时反馈给我们。 
                Email：m3@wecise.com
            </p>

        </el-main>
    </el-container>
</template>

<script>

export default {
    name: "Welcome"    
}
</script>

<style scoped>
    .el-container{
        background: #ffffff!important;
    }
    .el-image{
        width:30vw;
    }
</style>