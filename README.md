# One Data Model Registry

[One Data Model Registry](https://one-data-model.github.io/prototype-registry/)

### Run In Development - On Terminal
1. ```npm install```
2. ```npm audit fix```
   * (if needed)

<<<<<<< HEAD
3. ```npm run-script build```
4. ```npm run-script build:server```
5. ```npm run-script server```

Go to Localhost:9088 in browser

### Basic Table

Basic table usage

![basic](./docs/images/basic.png)

```
<template>
  <div class="base-demo" style="width: 400px">
    <vue-table-dynamic :params="params"></vue-table-dynamic>
  </div>
</template>

<script>
import VueTableDynamic from 'vue-table-dynamic'

export default {
  name: 'Demo',
  data() {
    return {
      params: {
        data: [
          ['Cell-1', 'Cell-2', 'Cell-3'],
          ['Cell-4', 'Cell-5', 'Cell-6'],
          ['Cell-7', 'Cell-8', 'Cell-9']
        ]
      }
    }
  },
  components: { VueTableDynamic }
}

</script>
```

### Border

Bordered table usage

- `border:`_`true`_ with border
- `border:`_`false`_ without border

![border](./docs/images/border.png)

```
<template>
  <div style="width: 600px">
    <vue-table-dynamic :params="params" ref="table"></vue-table-dynamic>
  </div>
</template>

<script>
import VueTableDynamic from 'vue-table-dynamic'
export default {
  name: 'Demo',
  data() {
    return {
      params: {
        data: [
          ['Index', 'Data1', 'Data2', 'Data3'],
          [1, 'b3ba90', '7c95f7', '9a3853'],
          [2, 'ec0b78', 'ba045d', 'ecf03c'],
          [3, '63788d', 'a8c325', 'aab418']
        ],
        header: 'row',
        border: true
      }
    }
  },
  components: { VueTableDynamic }
}
</script>
```

### Stripe

Striped rows

- `stripe:`_`true`_ striped
- `stripe:`_`false`_ unstriped

![stripe](./docs/images/stripe.png)

```
<template>
  <div style="width: 600px">
    <vue-table-dynamic :params="params" ref="table"></vue-table-dynamic>
  </div>
</template>

<script>
import VueTableDynamic from 'vue-table-dynamic'
export default {
  name: 'Demo',
  data() {
    return {
      params: {
        data: [
          ['Index', 'Data1', 'Data2', 'Data3'],
          [1, 'b3ba90', '7c95f7', '9a3853'],
          [2, 'ec0b78', 'ba045d', 'ecf03c'],
          [3, '63788d', 'a8c325', 'aab418'],
          [4, 'hf7y8c', '4rghjk', 'cgnhik']
        ],
        header: 'row',
        border: true,
        stripe: true
      }
    }
  },
  components: { VueTableDynamic }
}
</script>
```

### Highlight

Highlighted rows/columns/cells

- `highlight:`_`{row?:Array<number>; column?:Array<number>; cell?:Array<[number,number]>;}`_ configure highlighted rows, columns, cells. such as: _`{row: [1], column: [1], cell: [[-1, -1]]}`_ if negative, the position from the end of the array.
- `highlightedColor:`_`string`_ configure highlighted colors

![highlight](./docs/images/highlight.png)

```
<template>
  <div style="width: 600px">
    <vue-table-dynamic :params="params" ref="table"></vue-table-dynamic>
  </div>
</template>

<script>
import VueTableDynamic from 'vue-table-dynamic'
export default {
  name: 'Demo',
  data() {
    return {
      params: {
        data: [
          ['Index', 'Data1', 'Data2', 'Data3'],
          [1, 'b3ba90', '7c95f7', '9a3853'],
          [2, 'ec0b78', 'ba045d', 'ecf03c'],
          [3, '63788d', 'a8c325', 'aab418'],
          [4, 'hf7y8c', '4rghjk', 'cgnhik']
        ],
        header: 'row',
        border: true,
        stripe: true,
        highlight: { column: [-2] },
        highlightedColor: 'rgb(243, 235, 200)'
      }
    }
  },
  components: { VueTableDynamic }
}
</script>
```

### Multiple Select

Select multiple rows

- `showCheck:`_`boolean`_ show checkbox of rows
- `getCheckedRowDatas:`_`function`_ get data for all currently selected rows
- `setAllRowChecked:`_`function(selected:boolean)`_ toggle all selection
- `select:`_`event`_ currently selected/unselected rows

![checkbox](./docs/images/checkbox.png)

```
<template>
  <div style="width: 600px">
    <vue-table-dynamic
      :params="params"
      @select="onSelect"
      @selection-change="onSelectionChange"
      ref="table"
    >
    </vue-table-dynamic>
  </div>
</template>

<script>
import VueTableDynamic from 'vue-table-dynamic'
export default {
  name: 'Demo',
  data() {
    return {
      params: {
        data: [
          ['Index', 'Data1', 'Data2', 'Data3'],
          [1, 'b3ba90', '7c95f7', '9a3853'],
          [2, 'ec0b78', 'ba045d', 'ecf03c'],
          [3, '63788d', 'a8c325', 'aab418']
        ],
        header: 'row',
        showCheck: true
      }
    }
  },
  methods: {
    onSelect (isChecked, index, data) {
      console.log('onSelect: ', isChecked, index, data)
      console.log('Checked Data:', this.$refs.table.getCheckedRowDatas(true))
    },
    onSelectionChange (checkedDatas, checkedIndexs, checkedNum) {
      console.log('onSelectionChange: ', checkedDatas, checkedIndexs, checkedNum)
    }
  },
  components: { VueTableDynamic }
}
</script>
```

### Search

Filter rows by keyword

- `enableSearch:`_`boolean`_ enable/disable searching
- `search:`_`function(value:string)`_ manual row filtering
=======
`1. npm install`
>>>>>>> 6450010c501b41810e56496c0029ac928213df5e

`2. npm audit fix`
(if needed)

`3. npm run-script build`

`4. npm run-script build:server`

`5. npm run-script server`

Go to Localhost:9088 in browser
