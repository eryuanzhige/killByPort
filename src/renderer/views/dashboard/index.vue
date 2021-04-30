<template>
  <div class="dashboard-container">
    <el-row>
      <el-input style="width: 300px;" size="mini" v-model="port" clearable placeholder="请输入端口号"
                @keyup.enter.native="queryPort"></el-input>
      <el-button type="primary" size="mini" @click="queryPort">查询</el-button>
    </el-row>
    <el-row class="list-wrap">
      <el-row v-for="(item,index) of ports" :key="item[1]+index" class="item"
              @click.native="queryProgressInfo(item[4])">
        <el-col :span="4" class="i-col col-0">{{ item[0] }}(<span class="port">{{ item[5] }}</span>)</el-col>
        <el-col :span="5" class="i-col col-1">{{ item[1] }}</el-col>
        <el-col :span="6" class="i-col col-2">{{ item[2] }}</el-col>
        <el-col :span="4" class="i-col col-3">{{ item[3] }}</el-col>
        <el-col :span="4" class="i-col col-4">{{ item[4] }}</el-col>
        <el-col :span="1" class="i-col col-5">
          <i class="el-icon-close" @click="killProgress(item[4])"/>
        </el-col>
      </el-row>
    </el-row>
    <el-dialog
        title="占用进程明细"
        :visible.sync="dialogVisible"
        :show-close="false"
        width="60%">
      <div class="prog-list">
        <el-row v-for="(item,index) of progress" :key="item[1]+index" class="item">
          <el-col :span="16" class="i-col col-0">{{ item[0] }}</el-col>
          <el-col :span="8" class="i-col col-1">{{ item[4] }} {{ item[5] }}</el-col>
        </el-row>
      </div>
      <span slot="footer" class="dialog-footer">
            <el-button size="mini" type="danger" @click="killProgress(selPid)">关闭进程</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
const exec = require('child_process').exec
export default {
  name: 'dashboard',
  data() {
    return {
      port: '',
      dialogVisible: false,
      ports: [], // "TCP" "0.0.0.0:135" "0.0.0.0:0" "LISTENING" "1328"
      progress: [],
      selPid: ''
    }
  },
  created() {
    this.queryPort()
  },
  methods: {
    queryPort() {
      // 命令
      let cmdStr = 'netstat -ano'
      const port = this.port.trim()
      if (port && !isNaN(parseInt(port))) {
        cmdStr = 'netstat -aon|findstr ' + port
      }
      const workerProcess = exec(cmdStr, {})

      // 打印正常的后台可执行程序输出
      workerProcess.stdout.on('data', (data) => {
        if (data.indexOf('TCP') === -1) {
          this.ports = []
          return
        }
        const dataArr = data.split('\n')
        const filterArr = []
        dataArr.forEach(it => {
          const item = it.trim().split(/\s+/)
          if (item.length === 5 && !isNaN(parseInt(item[4]))) {
            // 取出端口号
            const ports = item[1].split(':')
            item.push(ports[ports.length - 1])
            filterArr.push(item)
          }
        })
        filterArr.sort((a, b) => a[5] - b[5])
        this.ports = filterArr
      })

      // 打印错误的后台可执行程序输出
      workerProcess.stderr.on('data', (data) => {
        console.log('stderr: ' + data)
        this.ports = []
        this.showNotification('失败', '没有找到任何端口', 'error')
      })

      // 退出之后的输出
      workerProcess.on('close', (code) => {
        if (code !== 0) {
          this.ports = []
          this.showNotification('失败', '没有找到任何端口', 'error')
        }
        console.log('out code：' + code)
      })
    },
    killProgress(pid) {
      const cmdStr = 'taskkill /pid ' + pid + ' -f'
      const workerProcess = exec(cmdStr, {})

      // 打印正常的后台可执行程序输出
      workerProcess.stdout.on('data', (data) => {
        console.log('success: ' + data)
      })

      // 打印错误的后台可执行程序输出
      workerProcess.stderr.on('data', (data) => {
        console.log('stderr: ' + data)
      })

      // 退出之后的输出
      workerProcess.on('close', (code) => {
        console.log('out code：' + code)
        this.showNotification('成功', '关闭进程成功', 'success')
        this.port = ''
        this.queryPort()
        this.dialogVisible = false
      })
    },
    queryProgressInfo(pid) {
      this.selPid = pid
      const cmdStr = 'tasklist|findstr ' + pid
      const workerProcess = exec(cmdStr, {})

      // 打印正常的后台可执行程序输出
      workerProcess.stdout.on('data', (data) => {
        console.log('success: ' + data)
        const dataArr = data.split('\n')
        const filterArr = []
        dataArr.forEach(it => {
          const item = it.trim().split(/\s+/)
          if (item.length === 6) {
            filterArr.push(item)
          }
        })
        this.progress = filterArr.slice(0, 5)
        if (dataArr.length === 0) {
          return
        }
        this.dialogVisible = true
      })

      // 打印错误的后台可执行程序输出
      workerProcess.stderr.on('data', (data) => {
        console.log('stderr: ' + data)
      })

      // 退出之后的输出
      workerProcess.on('close', (code) => {
        console.log('out code：' + code)
      })
    },
    showNotification(title = '通知', message = '', type = 'info') {
      this.$notify({
        type,
        title,
        message,
        position: 'top-right'
      })
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.dashboard {
  &-container {
    margin: 15px 10px;

    .list-wrap {
      height: calc(100vh - 80px);
      overflow: auto;
      padding: 0;
      margin-top: 20px;

      .item {
        padding-top: 10px;
        line-height: 40px;

        .i-col {
          color: #0376c2;
        }

        .col-0 {
          color: #777;
        }

        .col-1, .port {
          color: #0376c2;
        }

        .col-2 {
          color: #FF7F00;
        }

        .col-3 {
          color: #238E23;
        }

        .col-4 {
          color: #7F00FF;

          .el-icon-close {
            cursor: pointer;
          }
        }

        .col-5 {
          color: #f40021;

          .el-icon-close {
            cursor: pointer;
          }
        }
      }
    }
  }
}

.prog-list {
  .item {
    .col-0 {
      color: #0376c2;
    }

    .col-1 {
      color: #7F00FF;
    }
  }
}
</style>
