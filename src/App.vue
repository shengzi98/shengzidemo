<template>
  <div id="APP">
    <div class="containers" ref="content">
      <div class="canvas" ref="canvas"></div>
      <div id="js-properties-panel" class="panel"></div>
      <!-- 把操作按钮写在这里面 -->
      <div class="action">
        <el-upload action class="upload-demo" :before-upload="openBpmn">
          <el-button icon="el-icon-folder-opened"></el-button>
        </el-upload>
        <el-button class="new" icon="el-icon-circle-plus" @click="newDiagram"></el-button>
        <el-button icon="el-icon-download" @click="downloadBpmn"></el-button>
        <!--        <el-tooltip v-if="simulation" effect="light" :content="this.simulationStatus ? '退出模拟' : '开启模拟'">-->
        <!--          <el-button :size="headerButtonSize" :type="headerButtonType" icon="el-icon-cpu" @click="processSimulation">-->
        <!--            模拟-->
        <!--          </el-button>-->
        <!--        </el-tooltip>-->
        <a hidden ref="downloadLink"></a>
      </div>
    </div>
  </div>
</template>

<script>
//引入Bpmn建模器对象，主要通过创建这个对象创建建模器
import BpmnModeler from "bpmn-js/lib/Modeler";
import propertiesProviderModule from "bpmn-js-properties-panel/lib/provider/camunda";
import propertiesPanelModule from "bpmn-js-properties-panel";
import camundaModdleDescriptor from "camunda-bpmn-moddle/resources/camunda";
import {xmlStr} from "./mock/xmlStr";
// 汉化
import customTranslate from "./customTranslate";
export default {
  // 生命周期 - 创建完成（可以访问当前this实例）
  created() {},
// 生命周期 - 载入后, Vue 实例挂载到实际的 DOM 操作完成，一般在该过程进行 Ajax 交互
  mounted() {
    this.init()
  },
  data(){
    return{
      bpmnModeler:null,
      canvas:null,
      container: null,
      xmlUrl: '',
      defaultXmlStr: xmlStr
    };
  },
  methods:{
    getFilename(xml) {
      let start = xml.indexOf("process");
      let filename = xml.substr(start, xml.indexOf(">"));
      filename = filename.substr(filename.indexOf("id") + 4);
      filename = filename.substr(0, filename.indexOf('"'));
      return filename;
    },
    download({ name = "diagram.bpmn", data }) {
      // 这里就获取到了之前设置的隐藏链接
      const downloadLink = this.$refs.downloadLink;
      // 把数据转换为URI，下载要用到的
      const encodedData = encodeURIComponent(data);
      if (data) {
        // 将数据给到链接
        downloadLink.href =
          "data:application/bpmn20-xml;charset=UTF-8," + encodedData;
        // 设置文件名
        downloadLink.download = name;
        // 触发点击事件开始下载
        downloadLink.click();
      }
    },
    downloadBpmn() {
      this.bpmnModeler.saveXML({ format: true }, (err, xml) => {
        if (!err) {
          // 获取文件名
          const name = `${this.getFilename(xml)}.bpmn`;
          // 将文件名以及数据交给下载函数
          this.download({ name: name, data: xml });
        }
      });
    },
    openBpmn(file) {
      const reader = new FileReader();
      // 读取File对象中的文本信息，编码格式为UTF-8
      reader.readAsText(file, "utf-8");
      reader.onload = () => {
        //读取完毕后将文本信息导入到Bpmn建模器
        this.createNewDiagram(reader.result);
      };
      return false;
    },
    newDiagram(){
      this.createNewDiagram()
    },
    async init(){// 获取到属性ref为“content”的dom节点
      this.container = this.$refs.content
      // 获取到属性ref为“canvas”的dom节点
      const canvas = this.$refs.canvas
      // 将汉化包装成一个模块
      const customTranslateModule = {
        translate: ["value", customTranslate]
      };
      // 建模
      this.bpmnModeler = new BpmnModeler({
        container: canvas,
        //添加控制板
        propertiesPanel: {
          parent: '#js-properties-panel'
        },
        additionalModules: [
          // 左边工具栏以及节点
          propertiesProviderModule,
          // 右边的工具栏
          propertiesPanelModule,
          // 汉化模块
          customTranslateModule
        ],
        moddleExtensions: {
          camunda: camundaModdleDescriptor
        }
      })
      await this.createNewDiagram()
    },
    async createNewDiagram(){
      //将XML导入Bpmn-js建模器
      this.bpmnModeler.importXML(xmlStr,err =>{
        if(err){
          console.error("打开模型出错，请确认该模型符合Bpmn2.0规范");
        }else {
          // 这里是成功之后的回调, 可以在这里做一系列事情
          this.success()
        }
      });
    }
  },
  success () {
    // console.log('创建成功!')
    this.addEventBusListener()
  },
  addEventBusListener(){
    // 监听 element
    let that = this
    const eventBus = this.bpmnModeler.get('eventBus')
    const eventTypes = ['element.click', 'element.changed', 'element.updateLabel']
    eventTypes.forEach(function(eventType) {
      eventBus.on(eventType, function(e) {
        console.log(eventType)
        if (!e || e.element.type === 'bpmn:Process') return
        if (eventType === 'element.changed') {
          // that.elementChanged(e)
        } else if (eventType === 'element.click') {
          console.log('点击了element', e)
          // if (e.element.type === 'bpmn:Task') {
          // }
        } else if (eventType === 'element.updateLabel') {
          console.log('element.updateLabel', e.element)
        }
      })
    })
  }
}
</script>

<style scoped>
.containers{
  background-color: #ffffff;
  width: 100%;
  height: calc(100vh - 52px);
}
.canvas{
  width: 100%;
  height: 100%;
}
.panel{
  position: absolute;
  right: 0;
  top: 0;
  width: 300px;
}
.action {
  position: fixed;
  bottom: 10px;
  left: 10px;
  display: flex;
}
.upload-demo {
  margin-right: 10px;
}
</style>



