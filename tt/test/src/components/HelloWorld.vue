<template>
  <div>
    <TableDiy :columns='columns'
              :tableData='tableData'
              :excel="true">
    </TableDiy>
  </div>
</template>

<script>
import TableDiy from '@/components/TableDiy.vue'
export default {
  name: 'HelloWorld',
  data () {
    return {
      columns: [
        {
          title: '姓名',
          slot: 'name'
        },
        {
          title: '年龄',
          slot: 'age'
        },
        {
          title: '地址',
          slot: 'address'
        },
        {
          title: '一',
          slot: 'a'
        },
        {
          title: '二',
          slot: 'b'
        },
        {
          title: '三',
          slot: 'c'
        },
        {
          title: '四',
          slot: 'd'
        },
        {
          title: '五',
          slot: 'e'
        },
        {
          title: '六',
          slot: 'f'
        },
      ],
      tableData: [
        {
          name: 'John Brown',
          age: 18,
          address: 'New York No. 1 Lake Park',
          date: '2016-10-03',
          a: 'aregcwe',
          b: 'avtvdrs',
          c: 'adsfaewa',
          d: 'vvervrebr',
          e: 52342342,
          f: 6234234
        },
        {
          name: 'Jim Green',
          age: 24,
          address: 'London No. 1 Lake Park',
          date: '2016-10-01',
          a: 1345452,
          b: 'asdfaef',
          c: 'sd32fsdvds',
          d: 'avdvavasdv',
          e: 'asdvasdvsad',
          f: 'adva'
        },
        {
          name: 'Joe Black',
          age: 30,
          address: 'Sydney No. 1 Lake Park',
          date: '2016-10-02',
          a: 'zdsvzevz',
          b: '342342342',
          c: 32325,
          d: 'dsfvsv',
          e: 'vce3334efe',
          f: 's98s9d8vjj'
        },
        {
          name: 'Jon Snow',
          age: 26,
          address: 'Ottawa No. 2 Lake Park',
          date: '2016-10-04',
          a: 'viwjoe',
          b: 'vo8wejf9',
          c: 'moiefo8',
          d: '4f324',
          e: 23555,
          f: 684343
        }
      ]
    }
  },
  methods: {},
  components: {
    TableDiy
  }
}
</script>

<style>
</style>
