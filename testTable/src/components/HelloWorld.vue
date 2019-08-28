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
          key: 'name'
        },
        {
          title: '年龄',
          key: 'age'
        },
        {
          title: '地址',
          key: 'address',
          children: [
            {
              title: '二级地址',
              key: 'address2'
            },
            {
              title: '三级地址',
              key: 'address3'
            },
          ]
        },
        {
          title: '一',
          key: 'a'
        },
        {
          title: '二',
          key: 'b'
        },
        {
          title: '三',
          key: 'c'
        },
        {
          title: '四',
          key: 'd'
        },
        {
          title: '五',
          key: 'e'
        },
        {
          title: '六',
          key: 'f'
        },
      ],
      tableData: [
        {
          name: 'John Brown',
          age: 18,
          address2: 'New York',
          address3: 'No. 1 Lake Park',
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
          address2: 'London',
          address3: 'No. 1 Lake Park',
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
          address2: 'Sydney',
          address3: 'No. 1 Lake Park',
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
          address2: 'Ottawa',
          address3: 'No. 2 Lake Park',
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
