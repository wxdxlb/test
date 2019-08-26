<template>
  <div>
    <Table :columns="columns"
           :data="showData"
           @on-row-dblclick="handleEdit">

      <template v-for="(v,i) in columns"
                slot-scope="{ row, index }"
                :slot="v.slot">
        <Input type="text"
               v-model="edit[v.slot]"
               v-if="editIndex === index"
               :key="i"
               @on-enter="handleSave(index)" />
        <span v-else
              :key="i">{{ row[v.slot] }}</span>
      </template>

      <template slot-scope="{ row, index }"
                slot="action">
        <div v-if="editIndex === index">
          <Button @click="handleSave(index)">保存</Button>
          <Button @click="editIndex = -1">取消</Button>
        </div>
        <div v-else>
          <Button @click="handleEdit(row, index)">操作</Button>
        </div>
      </template>

    </Table>
    <template v-if="excel">
      <div style="padding:10px">
        导入表格
        <input id="upload"
               type="file"
               @change="importExcel"
               accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel" />
      </div>
      <div style="padding:10px">
        <button @click="outExcel">导出</button>
        <label><input type="checkbox"
                 v-model="daochu">是否按需导出字段（不选默认导出全部字段）</label>
        <ul v-show="daochu"
            style="padding:5px;list-style:none">
          <li v-for="(v,i) in columns"
              :key="i"
              style="display:inline-block;padding:5px">
            <label><input type="checkbox"
                     :value="v.slot"
                     v-model="outArr">{{v.title}}</label>
          </li>
        </ul>
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
      editIndex: -1,  // 当前聚焦的输入框的行数
      edit: {},  //编辑的数据
      daochu: false
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
          outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]);//outdata就是你想要的东西
          this.da = [...outdata]  //导入的数据
          const result = this.da.map(o => { //把title汉字转换为对应的slot
            let obj = {}
            _this.columns.map(v => {
              obj[v.slot] = o[v.title]
            })
            return obj
          });
          _this.showData = result //将数据赋值给表格数据，渲染页面
        }
        reader.readAsArrayBuffer(f);
      }
      reader.readAsBinaryString(f);
    },

    outExcel () { //用到的参数4个：选择性导出的字段Array；全部表头字段Array（包含title，slot）；导出表格内的数据；导出的表格名称；
      if (confirm('此操作将导出excel文件, 是否继续?')) {
        let _this = this;
        require.ensure([], () => {
          const { export_json_to_excel } = require('../excel/Export2Excel');
          let filterCol = _this.columns.filter(v => {
            return _this.outArr.includes(v.slot);
          })
          filterCol.length === 0 ? filterCol = _this.columns : null;  //过滤后是空数组，表示没有筛选条件，导出全部字段
          const excelHeader = filterCol.map(v => v.title); //导出的excel中显示的表头名字
          const filterVal = filterCol.map(v => v.slot); // 导出的表头字段名
          const list = _this.showData;//你要导出的数据list。
          const data = list.map(v => filterVal.map(j => v[j]))
          export_json_to_excel(excelHeader, data, `导出的excel`);// 导出的表格名称，根据需要自己命名
        })
      }
    },

    handleEdit (row, index) { //双击编辑表格某一行
      this.edit = { ...row }
      this.editIndex = index;
    },
    handleSave (index) {  //在input框空按回车，完成编辑
      this.showData.splice(index, 1, { ...this.edit })
      this.editIndex = -1;
    }
  }
}
</script>

<style scoped>
</style>
