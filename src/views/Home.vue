<template>
    <my-page title="十六进制查看" :page="page">
        <div class="editor-box" :style="{right: toolBoxWidth + 'px'}">
            <div class="editor-box-content">
                <input type="file" @change="fileChange($event)">
                <div class="loading" v-if="loading">读取中...</div>
                <div class="data-box" v-if="displayResult">
                    <div class="list">
                        <div class="list-row data-header">
                            <div class="grid grid-offset">偏移</div>
                            <div class="grid grid-data grid-data-header">
                                <div class="grid grid-ceil" v-for="item in headers">{{ item }}</div>
                            </div>
                            <div class="grid grid-asc">ASCII</div>
                        </div>
                        <div class="list-row-box">
                            <div class="list-row list-row-offset">
                                <div class="grid grid-offset" v-for="row, index in displayResult">{{ offset(index) }}</div>
                            </div>
                            <div class="list-row list-row-hex">
                                <div class="grid grid-data" v-for="row, index in displayResult">
                                    <div class="grid grid-ceil" v-for="item, column in row"
                                         :class="ceilClass(index, column)"
                                         @mousedown="ceilMouseDown($event, index, column)"
                                         @mousemove="ceilMouseMove($event, index, column)"
                                         @mouseup="ceilMouseUp($event, index, column)"
                                         @click="select(index, column)">{{ hex(item) }} </div>
                                </div>
                            </div>
                            <div class="list-row list-row-asc">
                                <div class="asc" v-for="row, index in displayResult">
                                    <span v-for="item, column in ascArr(row)">
                                        <span class="asc-ceil"
                                              :class="ascCeilClass(index, column)"
                                              @mousedown="ceilMouseDown($event, index, column)"
                                              @mousemove="ceilMouseMove($event, index, column)"
                                              @mouseup="ceilMouseUp($event, index, column)">{{ item }} </span>
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="btns">
                        <ui-raised-button class="btn" label="上一页" @click="prevPage" />
                        <ui-raised-button class="btn" label="下一页" @click="nextPage" />
                        <!--{{ dataOffset }}-->
                    </div>
                    <div>
                        <ui-slider class="slider" v-model="dataOffset" :step="1" :min="0" :display-value="false" :max="result.length" />
                    </div>
                </div>
            </div>
        </div>
        <div class="tool-box" :style="{width: toolBoxWidth + 'px'}">
            <div class="handler" @mousedown="handlerMouseDown($event)"></div>
            <div class="search-box">
                <input class="input" v-model="keyword" placeholder="搜索十六进制编码或 ASCII">
                <ui-icon-button icon="search" label="搜索" @click="search" />
            </div>
            <ui-article class="article">
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
                dataOffset: 0,
                toolBoxWidth: 256,
                // selection
                startPosition: 0,
                endPosition: 0,
                loading: false,
                headers: ['00', '01', '02', '03', '04', '05', '06', '07', '08', '09', '0A', '0B', '0C', '0D', '0E', '0F'],
//                result: null,
                displayResult: null,
                ceil: null,
//                ceil: 71,
                // search
                keyword: '',
//                page: {
//                    menu: [
//                        {
//                            type: 'icon',
//                            icon: 'language',
//                            to: '/ex',
//                            title: '语言'
//                        }
//                    ]
//                }
            }
        },
        computed: {
            ceilBinary() {
                return this.ceil.toString(2).padStart(8, '0')
            }
//            displayResult() {
//                console.log('计算')
//                return this.result.slice(this.dataOffset, this.dataOffset + 20)
//            }
        },
        mounted() {
            console.log('开始')
            console.log(doubleConversion.uint8(71))
            console.log('47'.toString(10))
            console.log(String.fromCharCode(71))
            this.init()
        },
        methods: {
            prevPage(e) {
                this.dataOffset -= 20
                if (this.dataOffset < 0) {
                    this.dataOffset = 0
                }
                this.loadPage()
            },
            nextPage(e) {
                console.log('下一页')
                this.dataOffset += 20
//                let start = new Date
                this.loadPage()
                console.log('数据完成', this.displayResult)
            },
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
            ceilMouseDown(e, row, column) {
                this.ceilDown = true
                let position = (this.dataOffset + row) * 16 + column
                this.startPosition = position
                this.endPosition = position
            },
            ceilMouseUp(e, row, column) {
                this.ceilDown = false
            },
            ceilMouseMove(e, row, column) {
                if (this.ceilDown) {
                    console.log(row, column)
                    let position = (this.dataOffset + row) * 16 + column
                    this.endPosition = position
                }
            },
            ceilClass(row, column) {
                let position = (this.dataOffset + row) * 16 + column
                if (position >= this.startPosition && position <= this.endPosition) {
                    return ['active']
                }
                return []
            },
            ascCeilClass(row, column) {
                let position = (this.dataOffset + row) * 16 + column
                if (position >= this.startPosition && position <= this.endPosition) {
                    return ['active']
                }
                return []
            },
            select(row, column) {
                let position = (this.dataOffset + row) * 16 + column
                this.startPosition = position
                this.endPosition = position
                this.ceil = this.displayResult[row][column]
            },
            ascArr(row) {
                return row.map(item => {
                    let ret = String.fromCharCode(item)
                    if (!ret && !ret.length) {
                        return '.'
                    }
                    return ret
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
                this.ascText = ''
                this.hexText = ''
                for (let i = 0; i < array.length; i++) {
                    if (i % 16 === 0) {
                        tempArr = []
                    }
                    this.ascText += String.fromCharCode(array[i])
                    this.hexText += array[i].toString(16).toUpperCase()
                    intArr.push(array[i])
//                    tempArr.push(array[i].toString(16))
                    tempArr.push(array[i])
                    if (i % 16 === 15) {
                        ret.push(tempArr)
                    }
                }
                console.log('长度' + this.ascText.length)
                console.log(this.hexText.substring(0, 100))
                ret.push(tempArr)
                console.log(ret)
                this.result = ret
                this.loadPage()
                this.loading = false
                let loadTime = ((new Date().getTime() - this.startTime) / 1000).toFixed(2)
                console.log(`加载了 ${loadTime}ms`)
            },
            search() {
                if (!this.keyword) {
                    this.$message({
                        type: 'danger',
                        text: '请输入关键词'
                    })
                    return
                }
                let idx = this.ascText.indexOf(this.keyword) // toUpperCase
                if (idx !== -1) {
                    console.log('找到了' + idx)
                    this.dataOffset = idx / 16
                    console.log(this.dataOffset)
                    this.loadPage()
                } else {
                    idx = this.hexText.indexOf(this.keyword.toUpperCase()) // TODO
                    if (idx !== -1) {
                        console.log('找到了' + idx)
                        this.dataOffset = idx / 16 / 2 + 1
                        console.log(this.dataOffset)
                        this.loadPage()
                    } else {
                        this.$message({
                            type: 'danger',
                            text: '搜索不到文本'
                        })
                        console.log('找不到')
                    }
//                    this.$message({
//                        type: 'danger',
//                        text: '搜索不到文本'
//                    })
//                    console.log('找不到')
                }
            },
            offset(index) {
                return ((this.dataOffset + index) * 16).toString(16).toUpperCase().padStart(8, '0')
            },
            init() {
            },
            loadPage() {
                this.displayResult = this.result.slice(this.dataOffset, this.dataOffset + 20)
            }
        },
        watch: {
            dataOffset() {
                this.loadPage()
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
        overflow: auto;
    }
    .editor-box-content {
        position: absolute;
        top: 0;
        left: 0;
        bottom: 0;
        width: 950px;
        padding: 16px;
    }
    .tool-box {
        position: absolute;
        top: 0;
        right: 0;
        width: 256px;
        bottom: 0;
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
        .article {
            padding: 16px;
        }
        table {
            width: 100%;
        }
    }
    .search-box {
        display: flex;
        input {
            height: 48px;
            flex-grow: 1;
            outline: none;
            border: none;
            padding-left: 16px;
        }
        border-bottom: 1px solid rgba(0,0,0,.12);
    }
    .list {
        height: 560px;
        margin-top: 24px;
        /*max-width: 640px;*/
    }
    .list-row-box {
        display: flex;
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
    .data-header {
        color: #999;
        /*text-align: center;*/
    }
    .list-row-offset {
        width: 80px;
    }
    .list-row-hex {
        width: 560px;
    }
    .list-row-asc {
        /*width: 560px;*/
    }
    .grid-offset {
        width: 80px;
        color: #999;
    }
    .asc-ceil {
        display: inline-block;
        width: 16px;
        text-align: center;
        cursor: pointer;
        &.active {
            background-color: #ccc;
        }
    }
    .loading {
        margin: 16px 0;
    }
    .btns {
        margin-top: 16px;
        margin-bottom: 16px;
        .btn {
            margin-right: 8px;
        }
    }
</style>
