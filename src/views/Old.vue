<template>
    <my-page title="十六进制编辑器">
        <div id="container">
            <div id="b">
                <input type="file"/>
                <input type="button" value="Up" id="up" class="scroll"/>
                <input type="button" value="Down" id="down" class="scroll"/>
                <input type="button" value="PgUp" id="pgup" class="scroll"/>
                <input type="button" value="PgDn" id="pgdn" class="scroll"/>
                <input type="button" value="Home" id="home" class="scroll"/>
                <input type="button" value="End" id="end" class="scroll"/>
            </div>
            <div id = "a">
                <table tabindex="1"></table>

                <table id="parser" contenteditable="true"></table>
            </div>
        </div>
    </my-page>
</template>

<script>
    /* eslint-disable */
    export default {
        data () {
            return {
            }
        },
        mounted() {
            this.init()
        },
        methods: {
            init() {
                function $c(tag){
                    return document.createElement(tag)
                }

                function $g(tag){
                    // get something
                    return document.querySelector(tag)
                }

                function $ga(tag){
                    // get something
                    return document.querySelectorAll(tag)
                }

                function parser(){
                    let tb = $g('#parser')
                    for(let i = 0; i < 5; i++){
                        let tr = $c('tr')
                        for(let j = 0; j < 3; j++){
                            let td = $c('td')
                            //td.contenteditable = 'true'
                            td.className = 'pp'
                            tr.appendChild(td)
                        }
                        tb.appendChild(tr)
                    }

                    document.addEventListener('keydown',function(){
                        console.log(event.target)
                        let c=event.target.className
                        if(c==='pp'){
                            console.log('ppppppppppppp')
                        }
                    })
                }

                function createTable(col, row, segment, offset){
                    let table = $g('table')

                    let ascii = true
                    //ascii = false

                    if(offset){
                        let tr = $c('tr')
                        if(segment){
                            let td = $c('td')
                            tr.appendChild(td)
                        }

                        for(let i = 0; i < col; i++){
                            let td = $c('td')
                            td.innerText = format_num(i, 2)
                            td.className = 'offset'
                            tr.appendChild(td)
                        }

                        if(ascii){
                            let td = $c('td')
                            td.colSpan = 16
                            td.innerHTML = 'ASCII'
                            tr.appendChild(td)
                        }
                        table.appendChild(tr)
                    }

                    for(let i = 0; i < row; i++){
                        let tr = $c('tr')

                        if(segment){
                            let row_head = $c('td')
                            row_head.className = 'segment'
                            tr.appendChild(row_head)
                        }
                        for(let j = 0; j < col; j++){
                            let td = $c('td')
                            td.className = 'hex'
                            tr.appendChild(td)
                        }

                        if(ascii){
                            for(let j = 0; j < col; j++){
                                let td = $c('td')
                                td.className = 'ascii'
                                tr.appendChild(td)
                            }
                        }
                        table.appendChild(tr)
                    }

                    return table
                }

                function randomHexArray(len){
                    let r = new Array(len).fill(0)
                    return r.map(x=>Math.floor(Math.random() * 256))
                }

                function format_num(num, width){
                    let fill0 = '0'.repeat(width)
                    return (fill0 + num.toString(16)).slice(-width).toUpperCase()
                }

                class Hex{
                    constructor(column, row){
                        this.COLUMN = column
                        this.ROW = row

                        this.table = createTable(this.COLUMN, this.ROW, true, true)
                        this.hex = this.table.querySelectorAll('.hex')
                        this.ascii = this.table.querySelectorAll('.ascii')
                        this.start = 0
                        this.action()
                        this.buffer = new Array(this.COLUMN * this.ROW).fill('')
                        this.mark = []


                        //TODO: 自定义键盘方案
                        // 预定义一组键，如-游戏键

                        this.drop_start = false

                        //TODO: VIM key_map
                        let vim_key_map = {}
                        this.key_map = {'j': 1, 'k': -1, 'ArrowDown': 'scroll_down', 'ArrowUp': 'scroll_up', 'PageDown': 'scroll_pgdn', 'PageUp': 'scroll_pgup', 'Home': 'scroll_home', 'End': 'scroll_end'}
                    }

                    load( data ){
                        this.data = data
                        this.mark = []
                    }

                    draw(){
                        //update buffer
                        if(this.data === undefined){
                            console.log('this.data undefined')
                            return
                        }

                        let buf_dat = this.data.slice(this.start, this.start + this.COLUMN * this.ROW)

                        //FIXME: 怎样不使用 uint8array 的 map
                        let buf_hex = []
                        for(let i = 0; i < buf_dat.length; i++){
                            buf_hex[i] = format_num(buf_dat[i], 2)
                        }

                        if(this.hex.length !== this.buffer.length){
                            console.error('tds.length !== this.buffer.length')
                        }

                        this.hex.forEach(x=>x.innerText = '')

                        for(let i = 0; i < buf_hex.length; i++){
                            //tds[i].innerText = this.buffer[i]
                            this.hex[i].innerText = buf_hex[i]
                        }

                        // draw ASCII
                        this.ascii.forEach(x=>x.innerText = '')
                        for(let i = 0; i < buf_hex.length; i++){
                            let ascii_code = parseInt(buf_hex[i], 16)
                            let c = String.fromCharCode( ascii_code )
                            this.ascii[i].innerText = /\w/.test(c)? c: '.'
                        }

                        let segment_ui = this.table.querySelectorAll('.segment')
                        if(segment_ui.length === 0){
                            return
                        }
                        // clear
                        segment_ui.forEach(x=>x.innerText = '')
                        for(let addr = this.start, i = 0; addr < this.start + buf_hex.length; addr += this.COLUMN, i++){
                            segment_ui[i].innerText = format_num(addr, 4)
                        }

                        this.draw_marks()
                    }

                    draw_marks(){
                        for(let i=0;i<this.hex.length; i++){
                            if(this.mark.includes(i + this.start)){
                                this.hex[i].style.background = 'yellow'
                                this.ascii[i].style.background = 'yellow'
                            }else{
                                this.hex[i].style.background = 'lightgray'
                                this.ascii[i].style.background = 'white'
                            }
                        }
                    }

                    draw_mark(td){
                        //console.log(event.target)
                        //let td = event.target
                        //if(td.className !== 'hex'){
                        if(!['hex','ascii'].includes(td.className)){
                            return
                        }

                        //TODO: indexOf() for object_array ?
                        let j = 0
                        //let cons = {'hex'}
                        let cons = this[td.className]
                        for(let i=0;i<cons.length;i++){
                            if(td === cons[i]){
                                j = i
                                break
                            }
                        }

                        let k = this.start + j
                        if(this.mark.includes(k)){
                            let ki = this.mark.indexOf(k)
                            this.mark.splice(ki, 1)
                            this.hex[j].style.background = 'lightgray'
                            this.ascii[j].style.background = 'white'
                        }else{
                            this.mark.push(k)
                            this.hex[j].style.background = 'yellow'
                            this.ascii[j].style.background = 'yellow'
                        }
                    }

                    drop_mark(){
                        if(this.drop_start === false){
                            return
                        }
                        let td = event.target
                        this.draw_mark(td)
                        //return
                    }

                    hex_over(){
                        let tgt = event.target
                        if(tgt.className === 'hex'){
                            let css = tgt.style
                            let bg = css.background
                            let fg = css.color
                            css.background = fg
                            css.color = bg
                        }
                    }
                    hex_out(){
                        let tgt = event.target
                        if(tgt.className === 'hex'){
                            tgt.style.background = this.save_hover
                        }
                    }


                    action(){
                        document.addEventListener('keydown', ()=>this.keyboard_scroll())
                        document.addEventListener('wheel', ()=>this.wheel_scroll())

                        document.addEventListener('mousedown', ()=>this.drop_start = true)

                        document.addEventListener('mouseover', ()=>this.drop_mark())
                        document.addEventListener('mouseup', ()=>this.drop_start = false)

                        // set all scroll button
                        let scroll_buttons = document.querySelectorAll('.scroll')
                        for(let button of scroll_buttons){
                            button.addEventListener('mousedown', ()=>this['scroll_' + button.id]())
                        }

                        document.addEventListener('mousedown',()=>this.draw_mark(event.target))

                        // test
                        //$g('table').addEventListener('mouseover', ()=>this.hex_over())
                        //$g('table').addEventListener('mouseout', ()=>this.hex_out())
                    }
                    //////////////////////////////////////////////////////////////
                    // Process Scroll
                    //////////////////////////////////////////////////////////////
                    keyboard_scroll(){
                        let fn = this.key_map[event.key] || 'empty_fn'
                        this[fn]()
                    }

                    empty_fn(){
                    }

                    wheel_scroll(){
                        if(event.deltaY > 0){
                            this.scroll_down()
                        }else{
                            this.scroll_up()
                        }
                    }

                    scroll_line(n){
                        // 滚动策略: 至少显示 1 行
                        let new_start = this.start + n * this.COLUMN
                        if(new_start < 0){
                            new_start = 0
                        }
                        if(new_start >= this.data.length){
                            return
                        }
                        this.start = new_start

                        this.draw()
                    }

                    scroll_up(){
                        this.scroll_line(-1)
                    }

                    scroll_down(){
                        this.scroll_line(1)
                    }

                    scroll_pgup(){
                        this.scroll_line(-this.ROW)
                    }

                    scroll_pgdn(){
                        this.scroll_line(this.ROW)
                    }

                    scroll_home(){
                        this.start = 0
                        this.draw()
                    }

                    scroll_end(){
                        this.start = this.data.length - this.data.length % this.COLUMN
                        this.draw()
                    }

                }

                function app() {
                    var input = document.querySelector('input')
                    input.addEventListener('change', function(e){
                        read_file(this.files[0], show_file)
                    })
                }

                function read_file(file, onload_callback){
                    var fr = new FileReader();
                    fr.onload = onload_callback
                    fr.readAsArrayBuffer(file);
                    //fr.readAsBinaryString(file);
                }

                function show_file(e){
                    let data = e.target.result
                    var array = new Uint8Array(data);
                    hex.load(array)
                    hex.draw()
                }

                var hex
/////////////////////////////////////////////////////////////
                app()
                hex = new Hex(16, 16)
                //parser()
                //test
                //1=2
                //return
                let d = randomHexArray(364)
                hex.load(d)
                hex.draw()
                //document.body.appendChild( hex.table )
            }
        }
    }
</script>

<style lang="scss" scoped>
table{
    border-collapse:collapse;
    margin-top: 2vh;
}
td{
    border:1px solid;
    width:5vw;
    max-width: 5vw;
    height:3vh;
    text-align: center;

    overflow: hidden;
    user-select:none;
}

.offset, .segment{
    width:6vw;
    max-width: 6vw;
    color: white;
    background: gray;
}

.ascii{
    border:none;
}
input[type="file"]{
    background: green;
}

input[type="button"]{
    width:8vw;
}

#container{
    border: solid 1px;
}
</style>
