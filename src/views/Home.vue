<template>
    <my-page title="十六进制查看" :page="page">
        <div class="editor-box" :style="{right: toolBoxWidth + 'px'}">
            <input type="file" @change="fileChange($event)">

            <div class="loading" v-if="loading">读取中...</div>

            <div class="list" v-if="result">
                <div class="list-row">
                    <div class="grid grid-offset">/</div>
                    <div class="grid grid-data grid-data-header">
                        <div class="grid grid-ceil" v-for="item in headers">{{ item }}</div>
                    </div>
                    <div class="grid grid-asc"></div>
                </div>
                <div class="list-row" v-for="row, index in result">
                    <div class="grid grid-offset">{{ offset(index) }}</div>
                    <div class="grid grid-data">
                        <div class="grid grid-ceil" v-for="item, column in row"
                             :class="ceilClass(index, column)"
                             @click="select(index, column)">{{ hex(item) }}</div>
                    </div>
                    <div class="asc">
                        <span v-for="item, column in ascArr(row)">
                            <span class="asc-ceil" :class="ascCeilClass(index, column)">{{ item }} </span>
                        </span>
                    </div>
                </div>
            </div>
        </div>
        <div class="tool-box" :style="{width: toolBoxWidth + 'px'}">
            <div class="handler" @mousedown="handlerMouseDown($event)"></div>
            <ui-article>
                <table v-if="ceil">
                    <tr>
                        <th>二进制（8位）</th>
                        <td>{{ ceilBinary }}</td>
                    </tr>
                    <tr>
                        <th>Int8</th>
                        <td>{{ ceil }}</td>
                    </tr>
                    <tr>
                        <th>UInt8</th>
                        <td>{{ ceil }}</td>
                    </tr>
                    <!--<tr>-->
                        <!--<th>Int16</th>-->
                        <!--<td>71</td>-->
                    <!--</tr>-->
                    <!--<tr>-->
                        <!--<th>UInt16</th>-->
                        <!--<td>71</td>-->
                    <!--</tr>-->
                    <!--<tr>-->
                        <!--<th>Int32</th>-->
                        <!--<td>71</td>-->
                    <!--</tr>-->
                    <!--<tr>-->
                        <!--<th>UInt32</th>-->
                        <!--<td>71</td>-->
                    <!--</tr>-->
                    <!--<tr>-->
                        <!--<th>Int64</th>-->
                        <!--<td>71</td>-->
                    <!--</tr>-->
                    <!--<tr>-->
                        <!--<th>UInt64</th>-->
                        <!--<td>71</td>-->
                    <!--</tr>-->
                </table>
            </ui-article>
        </div>
    </my-page>
</template>

<script>
    /* eslint-disable */
    var doubleConversion = require('double-conversion-decimal-binary')

    export default {
        data () {
            return {
                toolBoxWidth: 256,
                rowIndex: 0,
                columnIndex: 0,
                loading: false,
                headers: ['00', '01', '02', '03', '04', '05', '06', '07', '08', '09', '0A', '0B', '0C', '0D', '0E', '0F'],
                result: null,
                ceil: null,
//                ceil: 71,
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'language',
                            to: '/ex',
                            title: '语言'
                        }
                    ]
                }
            }
        },
        computed: {
            ceilBinary() {
                return this.ceil.toString(2).padStart(8, '0')
            }
        },
        mounted() {
            console.log('开始')
            console.log(doubleConversion.uint8(71))
            console.log('47'.toString(10))
            console.log(String.fromCharCode(71))
            this.init()
        },
        methods: {
            handlerMouseDown(e) {
                let downX = e.pageX
                let onMouseMove
                let originWidth = this.toolBoxWidth
                document.body.setAttribute('user-select', 'none')
                document.addEventListener('mousemove', onMouseMove = e => {
                    this.toolBoxWidth = originWidth - e.pageX + downX
                    if (this.toolBoxWidth < 256) {
                        this.toolBoxWidth = 256
                    }
                })
                document.addEventListener('mouseup', e => {
                    document.removeEventListener('mousemove', onMouseMove)
                    document.body.setAttribute('user-select', 'inherit')
                })
            },
            ceilClass(row, column) {
                if (row === this.rowIndex && column === this.columnIndex) {
                    return ['active']
                }
                return []
            },
            ascCeilClass(row, column) {
                if (row === this.rowIndex && column === this.columnIndex) {
                    return ['active']
                }
                return []
            },
            select(row, column) {
                this.rowIndex = row
                this.columnIndex = column
                this.ceil = this.result[row][column]
            },
            ascArr(row) {
                return row.map(item => {
                    return String.fromCharCode(item)
                })
            },
//            asc(row) {
//                return row.map(item => {
//                    return String.fromCharCode(item)
//                }).join(' ')
//            },
            fileChange(e) {
                this.loading = true
                this.result = null
                let file = e.target.files[0]
                let reader = new FileReader()
                this.startTime = new Date().getTime()
                reader.onload = e => {
//                    console.log(e.target.result)
                    let array = new Uint8Array(e.target.result)
                    console.log(array)
                    console.log(array[0].toString(16))
                    this.dealData(array)
                }
                reader.readAsArrayBuffer(file)
            },
            hex(item) {
                return item.toString(16).toUpperCase()
            },
            dealData(array) {
                let ret = []
                let intArr = []
                let tempArr = []
                for (let i = 0; i < array.length; i++) {
                    if (i % 16 === 0) {
                        tempArr = []
                    }
                    intArr.push(array[i])
//                    tempArr.push(array[i].toString(16))
                    tempArr.push(array[i])
                    if (i % 16 === 15) {
                        ret.push(tempArr)

                    }
                }
                console.log(ret)
                this.result = ret
                this.loading = false
                let loadTime = ((new Date().getTime() - this.startTime) / 1000).toFixed(2)
                console.log(`加载了 ${loadTime}ms`)
            },
            offset(index) {
                return (index * 16).toString(16).toUpperCase().padStart(8, '0')
            },
            init() {
            }
        }
    }
</script>

<style lang="scss" scoped>
    .editor-box {
        position: absolute;
        top: 0;
        left: 0;
        right: 256px;
        bottom: 0;
        padding: 16px;
    }
    .tool-box {
        position: absolute;
        top: 0;
        right: 0;
        width: 256px;
        bottom: 0;
        padding: 16px;
        border-left: 1px solid rgba(0,0,0,.12);
        .handler {
            position: absolute;
            top: 0;
            left: 0;
            bottom: 0;
            width: 5px;
            /*background-color: #ccc;*/
            cursor: col-resize;
        }
        table {
            width: 100%;
        }
    }
    .list {
        margin-top: 24px;
        /*max-width: 640px;*/
    }
    .list-row {
        margin-bottom: 4px;
        overflow: hidden;
    }
    .grid {
        float: left;
        /*display: inline-block;*/
        width: 32px;
    }
    .grid-ceil {
        text-align: center;
        cursor: pointer;
        /*border: 1px solid #eee;*/
        &.active {
            background-color: #ccc;
        }
    }
    .grid-data {
        width: 560px;
        max-width: 640px;
        margin-right: 0px;
        overflow: hidden;
    }
    .grid-data-header {
        color: #999;
    }
    .grid-offset {
        width: 80px;
        color: #999;
    }
    .asc-ceil {
        display: inline-block;
        width: 16px;
        text-align: center;
        &.active {
            background-color: #ccc;
        }
    }
    .loading {
        margin: 16px 0;
    }
</style>
