<template>
  <section class="content">
    <section class="edit-content">
      <div class="edit-list" v-for="(item, key) in data" :key="key">
        <h3 @click="copy(key)">场景编号：{{key}}</h3>  
        <div class="res" v-for="(res, i) in item.res" :key="i + 'res'">
          <span @click="changeUser($event, key, i)">【{{res.label}}】—— {{res.mark}}</span>
          <textarea type="text" 
            oninput="this.style.height = this.scrollHeight+'px'" 
            v-model="res.value" 
            @keydown="addConversation($event, key, 'res', i === (item.res.length - 1))">
          </textarea>
          <img class="delete-icon" @click="deleteConversation(key, 'res', i)" src="@/assets/delete.svg" alt="">
          <i v-if="i === (item.res.length - 1)" 
          @click="addConversation(null, key, 'res', true)">Ctrl / Cmd + Enter 键添加对话</i>
        </div>
        <div class="req" v-for="(req, j) in item.req" :key="j + 'req'">
          <div>
            <textarea type="text" 
            oninput="this.style.height = this.scrollHeight+'px'" 
            v-model="req.value"
            @keydown="addConversation($event, key, 'req', j === (item.req.length - 1))"></textarea>
            <input type="text" v-model="req.next" />
            <img class="delete-icon" @click="deleteConversation(key, 'req', j)" src="@/assets/delete.svg" alt="">
          </div>
          <i v-if="j === (item.req.length - 1)" 
          @click="addConversation(null, key, 'req', true)">Ctrl / Cmd + Enter 键添加对话</i>
        </div>
      </div>
      <button class="add-btn" type="button" @click="createScenes()"><img src="@/assets/add.svg">添加场景</button>
    </section>
    <section class="show-content">
      <pre>export default {{data}}</pre>
      <button class="copy-btn" stype="button" @click="copy(data)"><img src="@/assets/copy.svg">复制代码</button>
    </section>
    <textarea class="copy" type="text" ref="copy"></textarea>
    <transition name="slide-fade">
      <span class="toast" v-if="toastStatus">复制成功</span>
    </transition>
  </section>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

const createChatData = () => {
  return {
    label: '威威',
    mark: '我们的目标是远方的海',
    uid: 'weiwei',
    value: ''
  }
}

@Component
export default class Main extends Vue {
  // @Prop() private msg!: string
  private data: any = {}
  private toastStatus: boolean = false
  private changeData: any = {}

  private mounted() {
    this.createScenes()
  }
  private createTime() {
    return new Date().valueOf().toString()
  }

  private copy(value: any): void {
    let res = null
    if (typeof value === 'object') {
      res = JSON.stringify(value)
    } else {
      res = value
    }
    const input: any = this.$refs.copy
    input.value = res
    input.select()
    document.execCommand('copy')
    this.toastStatus = true
    setTimeout(() => {
      this.toastStatus = false
    }, 600)
  }

  private addConversation(e: any, key: string, user: string, bool: boolean) {
    const res = this.data
    this.updateData(res)
    if (!bool) {
      return
    }
    if (e) {
      // 快捷键添加
      if ((e.metaKey && e.keyCode === 13) || (e.ctrlKey && e.keyCode === 13)) {
        // Cmd + Enter || Ctrl + Enter
      } else {
        return
      }
    }
    // 点击添加
    switch (user) {
      case 'res':
        res[key].res.push(createChatData())
        break
      case 'req':
        res[key].req.push({
          value: '',
          next: key
        })
      default:
        break
    }
    this.updateData(res)
  }

  private deleteConversation(key: string, user: string, index: number) {
    const res = this.data
    switch (user) {
      case 'res':
        res[key].res.splice(index, 1)
        break
      case 'req':
        res[key].req.splice(index, 1)
        break
      default:
        break
    }
    this.updateData(res)
  }

  private createScenes(): void {
    const res: any = this.data
    const time = this.createTime()
    res[time] = {
      res: [],
      req: []
    }
    this.updateData(res)
    this.addConversation(null, time, 'res', true)
    this.addConversation(null, time, 'req', true)
  }

  private updateData(res: any): void {
    this.data = {}
    this.data = res
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
@import '../assets/styles/editList.less';

@bgColor: #a8d8b9;
.content {
  display: flex;
  height: 100%;
  box-sizing: border-box;
  .show-content {
    flex: 2;
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    padding: 20px;
    box-sizing: border-box;
    color: #36563c;
    background-color: @bgColor;
    background-image: url('../assets/texture.png');
    pre {
      flex: 1;
      overflow: auto;
      white-space: pre-wrap;
      word-wrap: break-word;
      width: 100%;
      &::-webkit-scrollbar {
        display: none;
      }
    }
    .copy-btn {
      background: #6a8372;
      color: #a8d8b9;
      width: 130px;
      height: 40px;
      border: none;
      font-size: 16px;
      border-radius: 2px;
      cursor: pointer;
      outline: none;
      display: flex;
      justify-content: center;
      align-items: center;
      box-shadow: 0 0 3px #36563c;
      img {
        margin-right: 5px;
      }
    }
  }
  .copy {
    position: absolute;
    z-index: 0;
    top: 0;
    opacity: 0;
  }
  textarea,
  input {
    border: none;
    height: auto;
    padding: 0.5em;
    border-radius: 2px;
    overflow: auto;
    outline: none;
    box-sizing: border-box;
    word-wrap: break-word;
    &:focus {
      box-shadow: 0 0 8px #006284;
    }
  }
  textarea {
    width: 400px;
    resize: none;
  }

  .slide-fade-enter-active {
    transition: all 0.3s ease;
  }
  .slide-fade-leave-active {
    transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
  }
  .slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active for below version 2.1.8 */ {
    transform: translateY(10px);
    opacity: 0;
  }
  span.toast {
    position: absolute;
    left: 50%;
    bottom: 15%;
    transform: translate(-50%, -50%);
    background: #6a8372;
    color: #a8d8b9;
    padding: 6px 14px;
    border-radius: 3px;
  }
}
</style>
