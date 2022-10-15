<template>
  <div id="app">
    <div v-for="(item, index) in msgs" :key="item.id">
      <div class="msg-container">
        <div class="msg-content">
          <span>{{item.content}} - 未读消息: {{item.count}}</span>     
        </div>
        <div class="badge-container">
        </div>
        <badge
          :id="index"
          :name="'cvs' + index"
          :count="item.count"
          @disappear="handleDisappear(index)">
        </badge>
      </div>
    </div>
  </div>
</template>

<script>
import Badge from './components/Badge.vue'

export default {
  name: 'App',
  components: {
    Badge
  },
  data () {
    return {
      msgs: []
    }
  },
  methods: {
    handleDisappear(index) {
      const element = this.msgs[index]
      element.count = 0
      this.$set(this.msgs, index, element)
    }
  },
  mounted() {
    let res = [];
    for(let i = 1;i < 3;i ++) {
      res.push({
        content: `这是第${i}条测试会话`,
        count: 99});
    }
    console.log(res)
    this.msgs = res
  }
}
</script>

<style>
* {
  margin: 0;
}
html, body, #app {
  width: 100%;
  height: 100%;
}
.msg-container {
  display: flex;
  justify-content: space-between;
  padding: 20px 30px 20px 30px;
  border-bottom: 1px solid rgb(223, 223, 223);
}
</style>
