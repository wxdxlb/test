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
      <!-- <div style="padding:10px">
        <button @click="outExcel">导出</button>
        <label><input type="checkbox"
                 v-model="daochu">是否按需导出字段（不选默认导出全部字段）</label>
        <ul v-show="daochu"
            style="padding:5px;list-style:none">
          <li v-for="(v,i) in columns"
              :key="i"
              style="display:inline-block;padding:5px">
            <label><input type="checkbox"
                     :value="v.key"
                     v-model="outArr">{{v.title}}</label>
          </li>
        </ul>
      </div> -->
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
      outArr: [],
      showData: this.tableData,
      daochu: false,
      allData: [],
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
    },
    daochu () {
      this.daochu ? null : this.outArr = []
    }
  },
  methods: {
    importExcel () {  //用到的参数：输入框事件对象获取当前文件；表头字段； //提供一个表格data接收数据
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

          //单行表头做法
          // outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]);//outdata就是你想要的东西
          // let tmpData = [...outdata]  //导入的数据
          // // console.log(tmpData);
          // const result = tmpData.map(o => { //把title汉字转换为对应的key
          //   let obj = {}
          //   _this.columns.map(v => {
          //     obj[v.key] = o[v.title]
          //   })
          //   return obj
          // });
          // _this.allData = result  //本地保存全部的数据
          // let showData = result.length > 10 ? result.slice(0, 10) : result; //渲染页面截取10条
          // _this.showData = showData //将数据赋值给表格数据，渲染页面
        }
        reader.readAsArrayBuffer(f);
      }
      reader.readAsBinaryString(f);
    },

    // outExcel () { //用到的参数4个：选择性导出的字段Array；全部表头字段Array（包含title，key）；导出表格内的数据；导出的表格名称；
    //   if (confirm('此操作将导出excel文件, 是否继续?')) {
    //     let _this = this;
    //     require.ensure([], () => {
    //       const { export_json_to_excel } = require('../excel/Export2Excel');
    //       let filterCol = _this.columns.filter(v => {
    //         return _this.outArr.includes(v.key);
    //       })
    //       filterCol.length === 0 ? filterCol = _this.columns : null;  //过滤后是空数组，表示没有筛选条件，导出全部字段
    //       const excelHeader = filterCol.map(v => v.title); //导出的excel中显示的表头名字
    //       const filterVal = filterCol.map(v => v.key); // 导出的表头字段名
    //       const list = _this.showData;//你要导出的数据list。
    //       // const list = _this.allData
    //       const data = list.map(v => filterVal.map(j => v[j]))
    //       export_json_to_excel(excelHeader, data, `导出的excel`);// 导出的表格名称，根据需要自己命名
    //     })
    //   }
    // },

    fuzadaochu () {
      require.ensure([], () => {
        const { export_table_to_excel } = require('../excel/Export2Excel');
        export_table_to_excel(this.columns, this.tableData) //可传4个参数，分别是表头数据、表格数据、导出的excel名，导出的sheet名
      })
    },

    //   fuzadaochu () {
    //     let XLSX = require('xlsx');
    //     let workbook = XLSX.utils.book_new()
    //     let tableSheet = XLSX.utils.table_to_sheet(document.getElementById('table'))
    //     this.trimData(tableSheet)
    //     console.log(tableSheet);
    //     return
    //     XLSX.utils.book_append_sheet(workbook, tableSheet, 'sheet1')
    //     let wbOut = XLSX.write(workbook, {
    //       bookType: 'xlsx',
    //       bookSST: true,
    //       type: 'array'
    //     })
    //     saveAs(new Blob([wbOut], { type: 'application/octet-stream' }), '复杂表头导出.xlsx')
    //   },
    //   trimData (obj) {
    //     for (var o in obj) {
    //       obj[o].v == '暂无筛选结果' ? delete obj[o] : null;
    //       if (typeof obj[o] == 'object') {
    //         this.trimData(obj[o])
    //       } else {
    //         if (typeof obj[o] == 'string') {
    //           obj[o] = obj[o].trim()
    //         }
    //       }
    //     }
    //   }
  }
}
</script>

<style scoped>
</style>
