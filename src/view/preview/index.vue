<template>
<div class="view-wrap">
  <dragCpnt :config="config" :editable="edit" :select="handleSelect" :initKey="handleInitKey"></dragCpnt>
</div>
</template>

<script>
import dragCpnt from './dragCpnt';
import request from '../../utils/request';
export default {
  components: {
    dragCpnt
  },

  data() {
    return {
      config: [],
      activeKey: '',
      drag: false,
      edit: false,
      dark: false
    }
  },

  computed: {
    dragOptions() {
      return {
        animation: 200,
        group: 'description',
        disabled: !this.edit,
        ghostClass: 'ghost'
      }
    }
  },

  created() {
    this.requestConfig();
  },

  mounted() {
    // this.config = this.$route.params.id && JSON.parse(this.$route.params.id) || [];
    // console.log(this.$route.params.id);
    window.addEventListener('message', this.recieveMsg);
    // this.config = config;
  },

  beforeDestroy() {
    window.removeEventListener('message', this.recieveMsg);
  },

  methods: {
    pushMsg(data) {
      window.top.postMessage(data);
    },
    recieveMsg({
      data
    }) {
      console.log('收到消息', data);
      let {
        type
      } = data;
      switch (type) {
        case 'addCpnt':
          this.handleAddCpnt(data.data);
          break;
        case 'getConfig':
          this.getConfig();
        case 'getPreview':
          this.saveConfig();
          break;
        case 'refreshConfig':
          this.requestConfig();
          break;
        case 'changeProps':
          this.handleChangeProps(this.activeKey, data.data);
          break;
        case 'delCpnt':
          this.handleDelCpnt(this.activeKey);
          break;
        case 'switchEdit':
          this.edit = data.data;
          break;
        case 'switchTheme':
          this.dark = data.data;
          break;
        case "buildPage":
          this.handleBuild();
          break;
        default:
          return;
      }
    },
    handleAddCpnt(item) {
      this.config.push(item);
    },

    getConfig() {
      console.log(this.config);
      this.pushMsg({
        type: 'getConfig',
        data: this.config
      });
    },

    saveConfig() {
      this.pushMsg({
        type: 'getPreview',
        data: this.config
      });
    },

    handleSelect(key, props) {
      if (!this.edit) return;
      this.activeKey = key;
      this.pushMsg({
        type: 'changeProps',
        data: props
      })
    },

    handleInitKey() {
      this.pushMsg({
        type: 'initKey',
        data: null
      })
    },

    handleChangeProps(key, props) {
      if (!key || !this.edit) return;
      this.config = this.changeDeepIndex(this.config, key, props);
    },

    handleDelCpnt(key) {
      if (!key || !this.edit) return;
      this.config = this.delDeepIndex(this.config, key)
      this.pushMsg({
        type: 'delCpnt',
        data: null
      })
    },

    changeDeepIndex: function (arr, key, res, prevKey = '') {
      return arr.map((item, index) => {
        let currentKey = prevKey !== '' ? prevKey + '-' + index : index.toString()
        if (currentKey === key) {
          item.props = res
        }
        if (item.children) item.children = this.changeDeepIndex(item.children, key, res, currentKey)
        return item
      })
    },

    delDeepIndex: function (arr, key, prevKey = '') {
      return arr.filter((item, index) => {
        let currentKey = prevKey !== '' ? prevKey + '-' + index : index.toString()
        if (currentKey === key) return false
        if (item.children && item.children.length > 0) item.children = this.delDeepIndex(item.children, key, currentKey);
        return true
      })
    },

    handleBuild() {
      console.log('跳路由', this.$router);
      localStorage.setItem('config', this.config);
      this.pushMsg({
        type: 'buildPage',
        data: this.config
      })
    },

    requestConfig() {
      let paramArray = window.top.location.pathname.split('/');
      let pid = paramArray[paramArray.length -1];
      console.log('route:', pid);
      request.get('/api/admin/page/details', {
        params: {
          pid
        }
      }).then(res => {
        console.log(res);
        let {
          code,
          data,
          message
        } = res;
        if (code == 0) {
          let tmp = JSON.parse(data.content);
          this.config = tmp;
          console.log('config', this.config);
        }
      }).catch(e => e);
    }
  }
}
</script>

<style lang="less">
body {
  margin: 0;
}
</style>
