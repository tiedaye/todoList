<template>
    <div :class="isShow?'tasks':''">
        <div class="tasks-container">
            <p class="tasks-title">This is my tasks ~</p>
            <div class="tasks-content">
                <div class="tasks-add">
                    <span @click="addTasks()">添加任务</span>
                    <span @click="searchTasks()">查找任务</span>
                    <span v-if="turn" @click="back()">返回</span>
                    <input type="text" class="tasks-input" placeholder="小主,请吩咐 ~" ref="addContent" style="outline: none">
                </div>
                <ul class="tasks-lists">
                    <transition-group appear tag="ul">
                        <li v-for="(value,idx) in list" :key="`${value.tasks}_${idx}`">
                            <!--<transition name="fade" v-for="(value,idx) in list" :key="`list_${idx}`">-->
                            <div :class="['tasks-details',value.showStyle?'selected':'']" >
                                <input type="checkbox" @click="changeBg(idx)" v-model="arr" :value="value">
                                <span :class="['tasks-detCon',value.showStyle?'line':'']">{{value.tasks}}</span>
                                <div>
                                    <span style="font-size:16px;padding: 1px 3px;background-color: #ccc;margin-right: 10px;border-radius:3px" @click="del(idx)">删除</span>
                                    <span style="font-size:16px;padding: 1px 3px;background-color: #ccc;border-radius:3px" @click="change(idx)">修改</span>
                                </div>
                            </div>
                            <!--</transition>-->
                        </li>
                    </transition-group>


                    <li style="font-size: 16px;margin-left: 20px">
                        {{msg}}已完成/共{{total}}个
                    </li>
                </ul>
            </div>
        </div>
        <div class="tasks-change" v-if="isShow">
            <div class="tasks-change-content">
                <div class="tasks-change-det">
                    <p>请输入要修改的内容:</p>
                    <input type="text" ref="changeContent">
                    <div class="tasks-change-btn">
                        <span @click="isShow=false">取消</span>
                        <span @click="complete()">完成</span>
                    </div>
                </div>
            </div>
        </div>
    </div>

</template>

<script>
    import proConf from '@/config/prod.js'
export default {
  data () {
      return{
          msg:0,
          total:3,
          arr:[],
          turn:false,
          isShow: false,
          storeList:[],
          list:[]
      }
  },
    methods:{
        getTasks () {
            this.$http.get(proConf.GET_TASKS).then(res => {
                console.log(res)
                this.list = res.data
            }).catch(e => {
                console.log(e)
           })
        },
        searchTasks () {
            if(!this.$refs.addContent.value){
                return
            }
            this.storeList = this.list
            this.turn = true
            var content = this.$refs.addContent.value
            this.list= this.list.filter((item)=>{
                return item.title == content
            })
            this.$refs.addContent.value = null
        },
        back () {
            this.list = this.storeList
            this.turn = false
        },
        change (xxx) {
            this.isShow = true
            this.msg = xxx
        },
        changeBg (item) {
            var arr = this.list
            arr[item].showStyle =!arr[item].showStyle
            if(arr[item].showStyle){
                this.msg++
            }else{
                this.msg--
            }
        },
      del (id) {
           this.list.splice(id,1)
      },
        addTasks (flag) {
            if(!flag){
                var content = this.$refs.addContent.value
                if(content){
                    var data = {'tasks':content,'showStyle':0}
                    this.$http.post(proConf.ADD_TASKS,this.$qs.stringify(data)).then((res)=>{
                        console.log(res)
                        this.list.push({'tasks':content,showStyle:0})
                    }).catch((e)=>{
                        console.log(e)
                    })
                    this.$refs.addContent.value = null
                    this.total++
                }else{
                    alert('请填写任务呦')
                }
            }else{
                this.complete()
            }


        },
        complete () {
            var b = this.$refs.changeContent.value
            if (b.match(/^[ ]*$/))
            {
                alert('不能为空');
            }else{
                var newArr = this.list
                newArr[this.msg].title = b
                this.isShow = false
            }

        }
    },
    created () {
      this.$nextTick(()=>{
          this.getTasks()
          document.onkeydown = (e) => {
              if(!e){
                  e = window.event
              }
              if((e.keyCode || e.which) == 13){
                      this.add(this.isShow)
              }
          }
      })
    }
}
</script>
<style scoped lang="scss">
.v-enter-active,.v-leave-active{
    transition: all .5s ease-in-out;
}
.v-enter,.v-leave-to{
    opacity: 0;
    transform: translateY(19.3px);
    }
.v-move{
    transition: all .5s ease-in-out;
}
.v-leave-active{
    width: 80%;
    position: absolute;
}
.tasks{
    height:100vh;
    overflow: hidden;
}
.tasks-container{
    width: 100%;
    .tasks-title{
        font-size: 24px;
        text-align: center;
        font-weight: 700;
    }
    .tasks-lists{
        width: 100%;
        margin-top: 50px;
        border:1px solid #ccc;
    }
    .tasks-content{
        width:80%;
        margin: 0 auto;
    }
    .tasks-add{
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 16px;
        span{
            border-radius:2px;
            padding: 1px 2px;
            background-color: aquamarine;
            margin-right: 20px;
        }
    }
    .tasks-input{
        font-size: 16px;
        border: .5px solid #ccc;
        border-radius: 10px;
        padding-left: 15px;
        width:50%;
        height:30px;
    }
    .tasks-details{
        height:45px;
        display: flex;
        justify-content: space-around;
        align-items: center;
        font-size: 18px;
        background-color: #feffee;
        border-bottom: 1px solid #ccc;
    }
    .tasks-detCon{
        display: inline-block;
        width:200px;
        overflow: hidden;
        white-space: nowrap;
        text-overflow:ellipsis;
    }
    .selected{
        background-color: wheat;
    }
    .line{
        color: forestgreen;
        text-decoration: line-through;
    }
}
    .tasks-change{
        width:100%;
        height:100%;
        position: fixed;
        top:0;
        left:0;
        background-color: rgba(0,0,0,.5);
    }
    .tasks-change-det{
        width: 70%;
    }
    .tasks-change-content{
        width:40%;
        height:40%;
        position: absolute;
        top:50%;
        left:50%;
        display: flex;
        justify-content: center;
        align-items: center;
        transform: translate(-50%,-50%);
        background-color: wheat;
        border-radius: 30px;
        p{
            text-align: center;
            font-weight: 700;
        }
        input{
            width:100%;
            height:25px;
            border: .5px solid #ccc;
            outline: none;
            border-radius: 10px;
            padding-left: 15px;
        }
        .tasks-change-btn{
            margin-top: 30px;
            display: flex;
            justify-content: space-around;
            span{
                display: inline-block;
                width:40px;
                height:20px;
                background-color: rebeccapurple;
                text-align: center;
                border-radius: 2px;
                color: #fff;
            }
        }
    }
</style>
