<template>
  <div class="dashboard-container">
    <el-row>
      <el-input style="width: 300px;" v-model="port" placeholder="请输入端口号" @keydown.enter="queryPort"></el-input>
      <el-button type="primary" @click="queryPort">查询</el-button>
    </el-row>
    <el-row class="list-wrap">
      <el-row v-for="(item,index) of ports" :key="item[1]+index" class="item">
        <el-col :span="4" class="i-col col-0">{{ item[0] }}</el-col>
        <el-col :span="5" class="i-col col-1">{{ item[1] }}</el-col>
        <el-col :span="6" class="i-col col-2">{{ item[2] }}</el-col>
        <el-col :span="4" class="i-col col-3">{{ item[3] }}</el-col>
        <el-col :span="4" class="i-col col-4">{{ item[4] }}</el-col>
        <el-col :span="1" class="i-col col-5">
          <i class="el-icon-close" @click="killProgress(item[4])"/>
        </el-col>
      </el-row>
    </el-row>
  </div>
</template>

<script>
const exec = require('child_process').exec
export default {
  name: 'dashboard',
  data() {
    return {
      port: '',
      ports: []// "TCP" "0.0.0.0:135" "0.0.0.0:0" "LISTENING" "1328"
    }
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
          return
        }
        const dataArr = data.split('\n')
        const filterArr = []
        dataArr.forEach(it => {
          const item = it.trim().split(/\s+/)
          if (item.length === 5 && !isNaN(parseInt(item[4]))) {
            filterArr.push(item)
          }
        })
        this.ports = filterArr
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
      padding: 20px 0;

      .item {
        padding-top: 10px;
        line-height: 40px;

        .i-col {
          color: #0376c2;
        }

        .col-0 {
          color: #777;
        }

        .col-1 {
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
          .el-icon-close{
            cursor: pointer;
          }
        }
        .col-5 {
          color: #f40021;
          .el-icon-close{
            cursor: pointer;
          }
        }
      }
    }
  }
}
</style>
