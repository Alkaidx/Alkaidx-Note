## 首先是题目
    实现一个LazyMan，可以按照以下方式调用:
    ```
    LazyMan("Hank")输出:
    Hi! This is Hank!
     
    LazyMan("Hank").sleep(10).eat("dinner")输出
    Hi! This is Hank!
    //等待10秒..
    Wake up after 10
    Eat dinner~
     
    LazyMan("Hank").eat("dinner").eat("supper")输出
    Hi This is Hank!
    Eat dinner~
    Eat supper~
     
    LazyMan("Hank").sleepFirst(5).eat("supper")输出
    //等待5秒
    Wake up after 5
    Hi This is Hank!
    Eat supper
    ```
    以此类推。

## 思路整理
    1. 链式调用
        TIPs： 在函数的return返回上下文，即return this
    2. 构建lazyman的类
        TIPs：在调用时new , 通过prototype写入方法
    3. 使用栈管理函数顺序执行
        TIPs： 可使用array
    4. 针对异步，使用观察者模式，通过订阅发布来执行
        TIPs: 在函数执行结束时通知观察者执行下一个函数

## 代码实战
    ```
    (function(window, undefined){
        // 声明一个类
        class Lazyman {
            constructor(name) {
                this.taskList = [];
            }
            // 订阅
            subscribe() {
                let task = {},
                    args = Array.from(arguments)
        
                if(args.length < 1) {
                    throw new Error('参数不能为空！')
                }
        
                task.name = args[0];
                task.args = args.slice(1);
                if(task.name === 'sleepFirstAction'){
                    this.taskList.unshift(task);
                }else{
                    this.taskList.push(task);
                }
            }
        
            // 发布
            publish(){
                if(this.taskList.length > 0){
                    this.run(this.taskList.shift());
                }
            }
            // 事件分发
            run(task){
                this[task.name](task.args)
            }
            //Log
            log(str){
                console.log(str);
            }
            // 命名
            lazyman(name){
                let str = 'Hi! This is' + name + '!';
                this.subscribe('lazymanAction',str);
                return this;
            }
            lazymanAction(str){
                this.log(str);
                this.publish();
            }
            // 吃
            eat(name){
                let str = 'Eat '+name+'~';
                this.subscribe('eatAction',str);
                return this;
            }
            eatAction(str){
                this.log(str);
                this.publish();
            }
            // 睡
            sleep(num){
                this.subscribe('sleepAction',num);
                return this;
            }
            sleepAction(num){
                setTimeout(()=>{
                    this.log("Wake up after "+ num);
                    this.publish();
                }, num*1000);
            }
            // 先睡
            sleepFirst(num){
                this.subscribe('sleepFirstAction',num);
                return this;
            }
            sleepFirstAction(num){
                setTimeout(()=>{
                    this.log("Wake up after "+ num);
                    this.publish();
                }, num*1000);
            }
        
        
        }
        window.LazyMan = function(str) {
            let lazyman = new Lazyman() 
            lazyman.lazyman(str)
            setTimeout(function(){
               lazyman.publish() 
            },0)
            return lazyman
        }
        
    })(window);
    ```