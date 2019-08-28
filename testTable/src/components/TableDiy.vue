<template>
  <div>
    <Table :columns="columns"
           :data="showData"
           id='table'>
    </Table>
    <template v-if="excel">
      <div style="padding:10px">
        复杂表格导入
        <input id="upload"
               type="file"
               @change="importExcel"
               accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel" />
      </div>
      <div style="padding:25px">
        <Button type="success"
                @click="fuzadaochu">复杂表头导出</Button>
      </div>
    </template>
  </div>
</template>

<script>
export default {
  props: {
    columns: {
      type: Array,
      require: true
    },
    tableData: {
      type: Array,
      require: true
    },
    excel: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      showData: this.tableData, //表格展示数据
      allData: [],  //储存全部数据
    }
  },
  computed: {
    len () {  //取得表头长度
      let offset = 0  //根据二级表头计算偏移量
      this.columns.forEach((v, i) => {
        if (v.children) {
          offset += (v.children.length - 1)
        }
      })
      return this.columns.length + offset
    },
    colObj () { //表格字段（展示数据时，每一列对应的字段）
      let c = {}
      this.columns.forEach(v => {
        c[v.key] = v.title
        if (v.children) {
          v.children.forEach(k => {
            c[k.key] = k.title
          })
        }
      })
      return c
    }
  },
  watch: {
    tableData () {
      this.showData = this.tableData
    }
  },
  methods: {
    importExcel () {  //用到的参数：输入框事件对象获取当前文件；表头字段
      let _this = this;
      // 通过DOM取文件数据
      let f = event.currentTarget.files[0];
      let reader = new FileReader();
      FileReader.prototype.readAsBinaryString = function (f) {
        let binary = "";
        let wb; //读取完成的数据
        let outdata;
        let reader = new FileReader();
        reader.onload = function (e) {
          let bytes = new Uint8Array(reader.result);
          let length = bytes.byteLength;
          for (let i = 0; i < length; i++) {
            binary += String.fromCharCode(bytes[i]);
          }
          let XLSX = require('xlsx');
          wb = XLSX.read(binary, {
            type: 'binary'
          });

          let excelData = wb.Sheets[wb.SheetNames[0]] //导入的原始数据
          let dealData = [] //处理导入后的数据
          for (var o in excelData) {
            if (Object.prototype.toString.call(excelData[o]) == '[object Object]') {
              dealData.push(excelData[o].v)
            }
          }
          dealData.pop()  //导入的数据结尾有个多余的undefined，删掉

          let d = []; //将导入的数据，按照字段个数分组
          for (let i = 0, len = dealData.length; i < len; i += _this.len) {
            d.push(dealData.slice(i, i + _this.len));
          }

          let th3 = d[1].map((v, i) => v ? v : d[0][i]) //合并后的表头，即表格数据的字段
          let th4 = th3.map(v => {  //处理后的表头字段
            for (var o in _this.colObj) {
              if (v == _this.colObj[o]) {
                return o
              }
            }
          })
          let data1 = d.slice(2)  //去掉前两行的一级二级表头
          let result = [] //转换完成的数据
          data1.map(v => {
            let obj = {}
            v.map((k, i) => {
              obj[th4[i]] = k
            })
            result.push(obj)
          })
          _this.allData = result  //本地保存全部的数据
          let showData = result.length > 10 ? result.slice(0, 10) : result; //渲染页面截取10条
          _this.showData = showData
        }
        reader.readAsArrayBuffer(f);
      }
      reader.readAsBinaryString(f);
    },

    fuzadaochu () {
      require.ensure([], () => {
        const { export_table_to_excel } = require('../excel/Export2Excel');
        export_table_to_excel(this.columns, this.tableData) //可传4个参数，分别是表头数据、表格数据、导出的excel名，导出的sheet名
      })
    }
  }
}
</script>

<style scoped>
</style>
