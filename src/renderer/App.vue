<template>
  <div id="app">
    <router-view></router-view>
    <el-dialog
        title="生活不易，吃碗酸辣粉吧"
        :visible.sync="dialogVisible"
        :modal="false"
        :show-close="false"
        top="3vh"
        width="60%">
      <div style="text-align: center">
        <img :src="aboutImg">
      </div>
      <span slot="footer" class="dialog-footer">
<!--    <el-button @click="dialogVisible = false">取 消</el-button>-->
        <!--    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>-->
  </span>
    </el-dialog>
  </div>
</template>

<script>
import aboutImg from '@/assets/about.gif'

export default {
  name: 'App',
  data() {
    return {
      aboutImg: aboutImg,
      dialogVisible: false
    }
  },
  created() {
    const { remote } = require('electron')
    const { Menu } = remote

    const dockMenu = Menu.buildFromTemplate([
      {
        label: '关于', click: () => {
          this.dialogVisible = true
        }
      }
    ])

    Menu.setApplicationMenu(dockMenu)
  }
}
</script>
<style lang="scss">
@import './styles/index.scss'; // 全局自定义的css样式
</style>
