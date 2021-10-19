<template>
  <div id="app">
        <div class="calc">
            <section class="calc__display">
                <div class="calc__display_content">
                    <div>{{lastMsg}}</div>
                </div>
                <div class="calc__display_result">
                    {{numberLength}}
                </div>
            </section>
            <div class="calc__mode" >
                <button class="calc__button" @click="press">7</button>
                <button class="calc__button" @click="press">8</button>
                <button class="calc__button" @click="press">9</button>
                <button class="calc__button" @click="press">*</button>

                <button class="calc__button" @click="press">4</button>
                <button class="calc__button" @click="press">5</button>
                <button class="calc__button" @click="press">6</button>
                <button class="calc__button" @click="press">/</button>

                <button class="calc__button" @click="press">1</button>
                <button class="calc__button" @click="press">2</button>
                <button class="calc__button" @click="press">3</button>
                <button class="calc__button" @click="press">-</button>

                <button class="calc__button" @click="press">0</button>
                <button class="calc__button" @click="press">.</button>
                <button class="calc__button" @click="press">00</button>
                <button class="calc__button" @click="press">+</button>

                <button class="calc__button" @click="press">C</button>
                <button class="calc__button" @click="press">Del</button>
                <button class="calc__button equal-sign" @click="press">=</button>
            </div> 
        </div>

    </div>
</template>

<script>
  export default {
    name: 'App',
    data() {
        return {
            msg: '0',
            lastMsg: '0',
            result: 0,
            isStart: false,
            lastArr: [],
            resultRecord: '',
            lastSymbol: '',
            changeMode: true
        }
    },
    computed: {
        numberLength() {
            let arr = String(this.msg).split('')
            if (arr.indexOf('.') === -1) {//没有输入小数点
                for (let i = 0; i < arr.length; i++) {
                    if (i === 3 || i === 7) {
                        arr.splice(1, 0, ',')
                    }
                    if (i > 4 && i !== 7) {
                        arr.splice(i - 4, 1)
                        arr.splice(i - 3, 0, ',')
                    }
                    if(i > 8) {
                        arr.splice(i - 8, 1)
                        arr.splice(i - 7, 0, ',')
                    }
                }
            } else {//有输入小数点的情况
                let dotIndex = this.msg.indexOf('.')
                let leftNum = arr.slice(0, dotIndex)
                let leftLength = leftNum.length
                if(leftLength > 3 && leftLength < 6) {
                    arr.splice(leftLength - 3, 0, ',')
                } else if (leftLength > 6) {
                    arr.splice(leftLength - 6, 0, ',')
                    arr.splice(leftLength - 2, 0,)
                } else {
                    return arr.splice(0,11).join('')            
                }
            }
            return arr.splice(0,11).join('')            
        }
    },
    methods: {
        press(key) {
            let val = key.target.textContent
            if (val === 'C') { // 清空所有数据,初始化所有数据
                this.msg = '0'
                this.lastMsg = '0'
                this.isStart = false
                this.lastArr = []
                this.resultRecord = ''
                this.result = 0
                return
            }

            if (this.numberLength.length === 11 && !isNaN(val)) { //限制最大输入长度
                return
            }
            if (!isNaN(val) || val === '.') { //输入的是数字或者小数点的情况
                if (!this.isStart) { // 如果没有开始，就把msg还原
                    this.msg = '0'
                }
                this.isStart = true
                if (this.isStart) { //第一次输入数字，要先把默认的0去掉 如果输入的首位为小数点的话，首位自动补零
                
                    if (this.msg.indexOf('0') === 0 && val === '.' && String(this.msg).indexOf('.') === -1) {
                         this.msg += val
                    } else if (this.msg.indexOf('0') === 0 && this.msg.indexOf('.') === -1) {
                        this.msg = this.msg.substring(1, this.msg.length)
                    }
                }
                if (String(this.msg).indexOf('.') !== -1) { //输入内容已经有小数点
                    if (val !== '.') { //如果输入不是点的情况，否则就没有输出
                        this.msg += val
                    }
                } else { //没有点的情况，直接拼接字符串
                    this.msg += val
                }

            } else { //输入符号的情况
                if (val === 'Del') { // 退格,在输出结果之后不能退格
                    if (!this.isStart) { //刚开始的时候不能输入符号
                        return
                    }
                    this.msg = this.msg.substring(0, this.msg.length - 1)
                    return
                }
                if (val !== '=') { //是符号但不是等号的情
                    if (!this.isStart) { //刚开始的时候不能输入符号
                        return
                    }
                    this.lastMsg = this.numberLength + val 
                    this.lastArr[this.lastArr.length] = this.lastMsg

                    if (this.lastArr.length > 1) { //第二次输入符号的时候也可以进行计算
                        if (this.resultRecord === '') { //第一次求值的情况
                            this.result = eval((this.lastArr[this.lastArr.length - 2]).replace(/,/g,'') + (this.numberLength).replace(/,/g,''))
                            this.resultRecord = this.result
                            this.lastMsg = String(this.result + val)
                            this.msg = String(this.result)
                        } else { //第二次求值，可以使用上一次求值结果直接计算
                            this.result = eval(this.resultRecord + this.lastSymbol + (this.numberLength).replace(/,/g,''))
                            this.resultRecord = this.result
                            this.lastMsg = String(this.result + val)
                            this.msg = String(this.result)
                        }
                        this.lastSymbol = val 
                        this.isStart = false
                    } else {
                        this.msg = '0' //清屏
                    }
                } else { //是等号的情况
                     if (this.resultRecord === '') { //第一次求值的情况
                        this.result = eval((this.lastMsg).replace(/,/g,'') + (this.numberLength).replace(/,/g,''))
                        this.resultRecord = this.result
                        this.msg = String(this.result)
                    } else { //第二次求值，可以使用上一次求值结果直接计算
                        this.result = eval(this.resultRecord + this.lastSymbol + (this.numberLength).replace(/,/g,''))
                        this.resultRecord = this.result
                        this.msg = String(this.result)
                    }
                    this.lastArr = [] //点击等于号之后清除lastArr，还原状态
                    this.resultRecord = '' //点击等于号之后清除resultRecord，还原状态
                }
            } 
        },
    },
  }

</script>

<style >
  body {
    background: whitesmoke;
  }
  
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    text-align: center;
  }
  
  .calc {
    width: 320px;
    height: 600px;
    padding: 20px;
    border-radius: 10px;
    margin: 20px auto;
    font-size: 16px;
    background-color: #e0e0e0;
    box-shadow: 8px 8px 14px #a4a4a4;
  }
  
  .calc__display {
    border-bottom: 1px solid #eee;
    height: 40%;
    display: flex;
    flex-direction: column;
  }
  .calc__display_content {
    position: relative;
    flex-grow: 1;
    text-align: right;
    word-break: break-all;
    font-size: 24px;
    color: #757575;
  }
  .calc__display_content div {
    position: absolute;
    right: 0;
    bottom: 0;
  }

  .calc__display_result {
    display: inline-block;
    padding: 8px 0;
    height: 36%;
    text-align: right;
    font-size: 52px;
    overflow: hidden;
    outline: none;
    color: #414141;
    word-break: break-all;
  }

  .calc__mode {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-evenly;
  }

  .calc__button {
    margin: 5px;
    width: 60px;
    height: 60px;
    outline: none;
    border: 0px;
    border-radius: 50%;
    color: #242323b9;
    cursor: pointer;
    box-shadow:  8px 8px 14px #a4a4a4,
                -8px -8px 14px #ffffff;
  }
 .calc__button:active {
    transform: translateY(2px)
  }

 .calc__button:hover {
    box-shadow: 14px 14px 14px #a4a4a4,
               -10px -10px 14px #ffffff;
  }
    
  .equal-sign {
    width: 125px;
    border-radius: 50%;
  }



</style>
