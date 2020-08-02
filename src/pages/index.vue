<template>
  <div>
    <Steps :current="current">
        <Step   v-for="(item,index) in step" :key="index" :title="item.title" :content="item.content"></Step>
    </Steps>
    <div class="home">
      <div class="left">
        <Form ref="formOne" :model="formOne">
          <Divider>连接mqttborker</Divider>
          <FormItem prop="serviceUri">
            <Input type="text" v-model="formOne.serviceUri" placeholder="地址"></Input>
          </FormItem>
          <FormItem prop="clintId">
            <Input type="text" v-model="formOne.clientId" placeholder="客户端ID"></Input>
          </FormItem>          
          <FormItem prop="userName">
            <Input type="text" v-model="formOne.userName" placeholder="用户名"></Input>
          </FormItem>
          <FormItem prop="password">
            <Input type="text" v-model="formOne.password" placeholder="密码"></Input>
          </FormItem>
          <FormItem>
            <Button type="primary" @click="onPublishClient('connect')" :disabled="formOne.disabledConnect">连接</Button>
            <Button type="primary" @click="onPublishClient('end')" :disabled="formOne.disabledClose">断开</Button>
            <Input type="text" v-model="formOne.msg" readonly></Input>
          </FormItem>
        </Form>
        <Form ref="formTwo" :model="formTwo">
          <Divider>模拟设备上报信息</Divider>
          <FormItem prop="userId">
            <Input type="text" v-model="formTwo.userId" placeholder="userId"></Input>
          </FormItem>
          <FormItem prop="productKey">
            <Input type="text" v-model="formTwo.productKey" placeholder="productKey"></Input>
          </FormItem>
          <FormItem prop="topic">
            <Input type="text" v-model="formTwo.topic" placeholder="topic"></Input>
          </FormItem>
          <FormItem label="data">
            <Input v-model="formTwo.data" type="textarea" placeholder="data"></Input>
          </FormItem>
          <FormItem>
            <Button type="primary" @click="publishTopic()">发布</Button>
          </FormItem>
        </Form>
      </div>
      <div class="right">
        <Form ref="formThree" :model="formThree">
          <Divider>连接mqttborker</Divider>
          <FormItem prop="serviceUri">
            <Input type="text" v-model="formThree.serviceUri" placeholder="地址"></Input>
          </FormItem>
          <FormItem prop="clintId">
            <Input type="text" v-model="formThree.clientId" placeholder="客户端ID"></Input>
          </FormItem>            
          <FormItem prop="userName">
            <Input type="text" v-model="formThree.userName" placeholder="用户名"></Input>
          </FormItem>
          <FormItem prop="password">
            <Input type="text" v-model="formThree.password" placeholder="密码"></Input>
          </FormItem>
          <FormItem>
            <Button type="primary" @click="onSubscribeClient('connect')" :disabled="formThree.disabledConnect">连接</Button>
            <Button type="primary" @click="onSubscribeClient('end')" :disabled="formThree.disabledClose">断开</Button>
            <Input type="text" v-model="formThree.msg" readonly></Input>
          </FormItem>
        </Form>
        <Form ref="formFour" :model="formFour">
          <Divider>订阅</Divider>
          例：第三方服务: /sdt/service/data/down <br/>物模型: /sdt/iot/command/down/a1zgZCfquML/0331583700288880000002
          <FormItem prop="topic">
            <Input type="text" v-model="formFour.topic" placeholder="主题"></Input>
          </FormItem>
          <FormItem>
            <Button type="primary" @click="subscribeTopic()">订阅</Button>
          </FormItem>
        </Form>
      </div>
    </div>
    <div class="home">
        <div class="left">
            <Divider>发送日志</Divider>
            <Input v-model="publishLog" type="textarea" placeholder="发送日志" readonly></Input>    
        </div>
        <div class="right">
            <Divider>接收日志</Divider>
            <Input v-model="subscribeLog" type="textarea" placeholder="接收日志" readonly></Input>  
        </div>
    </div>
  </div>
</template>

<script>
import mqtt from "mqtt";
import axios from "axios";
import uuidv1 from 'uuid/v1';
export default {
  data() {
    return {
      mqttClentuuId: "",
      publishLog: "",
      subscribeLog: "",
      client: null,
      publishClient: null,
      subscribeClient: null,
      formOne: {
        serviceUri: "ws://172.28.17.75:8083/mqtt",
        clientId: "0331583700288880000001",
        userName: "a1zgZCfquML//0331583700288880000001",
        password: "",
        disabledConnect: false,
        disabledClose: true,
      },
      formTwo: {
        userId: "0331583700288880000001",
        productKey: "a1zgZCfquML",
        topic: "/sdt/iot/data/up/event",
        data: "{\"image_inversion\":\"1\",\"event_type\":\"info\",\"service_id\":\"setting\",\"action\":\"event\",\"event_name\":\"properties\"}",
      },
      formThree: {
        serviceUri: "ws://172.28.17.75:8083/mqtt",
        clientId: "0331583700288880000002",
        userName: "a1zgZCfquML//0331583700288880000002",
        password: "",
        disabledConnect: false,
        disabledClose: true,
      },
      formFour: {
        // topic: "/sdt/iot/command/down/a1zgZCfquML/0331583700288880000002",
        topic: "/sdt/service/data/down",
      },
      current:1,
      step:[{
          title:"已完成",
          content:"这里是该步骤的描述信息"
      },
      {
          title:"进行中",
          content:"这里是该步骤的描述信息"
      },{
          title:"待进行",
          content:"这里是该步骤的描述信息"
      },{
          title:"待进行",
          content:"这里是该步骤的描述信息"
      },
      ]
    };
  },
  mounted() {
    // this.init();
    // axios.get('https://api.coindesk.com/v1/bpi/currentprice.json')
    // .then(response => (
    //     console.log(response)

    // ))
    this.init();
    
  },
  created() {
      this.mqttClentuuId = uuidv1(); //获取随机id
      this.logWsId = uuidv1();
  },
  methods: {
    init() {
      console.clear();
      var ws = this.websocket(1);
    },
    connectManager(client,clientId,connectUrl,username,password,msg,type){
      // 连接选项
      const options = {
        clean: true, // 保留回话
        connectTimeout: 60000, // 超时时间
        // 认证信息
        clientId: clientId,
        username: username,
        password: password,
        reconnectPeriod: 0,
      };

      console.info(connectUrl);
      client = mqtt.connect(connectUrl, options);
      client.on("reconnect", (error) => {
        console.log("正在重连:", error);
      });

      client.on("error", (error) => {
        console.log("连接失败:", error);
      });

      client.on("close", () => {
        var message = "连接被关闭，可能是clientId:"+clientId+"被挤占";
        if(type == "p"){
          this.formOne.msg = message;
          this.formOne.disabledConnect = false;
          this.formOne.disabledClose = true;
        }else if(type = "s"){
          this.formThree.msg = message;
          this.formThree.disabledConnect = false;
          this.formThree.disabledClose = true;
        }
      });

      client.on("message", (topic, message) => {
        this.subscribeLog += "\n";
        this.subscribeLog += message.toString();
        console.log("收到消息：", topic, message.toString());
      });

      client.on("connect", () => {        
        console.info(client);
        if(type == "p"){
          this.formOne.msg = msg;
          this.formOne.disabledConnect = true;
          this.formOne.disabledClose = false;
        }else if(type = "s"){
          this.formThree.msg = msg;
          this.formThree.disabledConnect = true;
          this.formThree.disabledClose = false;
        }
        console.log("建立连接成功");
      });

      client.on("end", () => {        
        if(type == "p"){
          this.formOne.msg = "断开连接成功";
          this.formOne.disabledConnect = false;
          this.formOne.disabledClose = true;
        }else if(type = "s"){
          this.formThree.msg = "断开连接成功";
          this.formThree.disabledConnect = false;
          this.formThree.disabledClose = true;
        }
        console.log("断开连接成功");
      });      
      return client;
    },
    websocket (sid) { 
        let ws = new WebSocket('ws://172.28.17.74:18889/serverlog/'+sid)
        ws.onopen = () => {
        // Web Socket 已连接上，使用 send() 方法发送数据
            console.log('数据发送中...')
            // ws.send('{"id":1,"command":"tail -f /opt/ltserver/log/skyworth-southbound.out | grep \'RuleEngine - 执行流程 - '+this.formTwo.productKey+' - '+this.formTwo.userId+'\'"}');
            ws.send('{"id":1,"command":"tail -f /opt/ltserver/log/skyworth-southbound.out | grep \'RuleEngine - 执行流程 - '+this.formTwo.productKey+' - '+this.formTwo.userId+'\'"}');
            // ws.send('{"id":1,"command":"tail -f /opt/ltserver/log/skyworth-southbound.out | grep 1"}');
            // ws.send('{"id":1,"command":"tail -f /opt/ltserver/log/skyworth-southbound.out"}');
        }
        // }
        ws.onmessage = evt => {
          //console.log('数据已接收...',evt.data)          
          this.step = [];
          var msg = evt.data;
          var mark = " - ";
          var start = msg.lastIndexOf(" - ");
          msg = msg.substring(start+mark.length, msg.length+1);
          console.info(msg);
        }
        ws.onclose = function () {
        // 关闭 websocket
          console.log('连接已关闭...')
        }
      return ws;
    },    
    onPublishClient(type){
      if(type == "connect"){
        this.publishClient = this.connectManager(this.publishClient,this.formOne.clientId,this.formOne.serviceUri,this.formOne.userName,this.formOne.password,"上行连接成功","p");        
      }else{
        this.publishClient.end(true);
      }
    },
    onSubscribeClient(type){
      if(type == "connect"){
        this.subscribeClient = this.connectManager(this.subscribeClient,this.formThree.clientId,this.formThree.serviceUri,this.formThree.userName,this.formThree.password,"下行连接成功","s");
      }else{
        this.subscribeClient.end(true);
      }
    },
    subscribeTopic(){
      console.info(this.subscribeClient);
      if(this.subscribeClient == null || this.subscribeClient.connected == false){
        this.$Message.info("请先建立连接!!!");
        return;
      }
      this.subscribeClient.subscribe(this.formFour.topic);
      this.$Message.info('订阅成功');
    },
    publishTopic(){
      if(this.publishClient == null || this.publishClient.connected == false){
        this.$Message.info("请先建立连接!!!");
        return;
      }
      var sendData = {
          "appName":"OTT",
          "data": this.formTwo.data,
          "messageId": uuidv1(),
          "productKey": this.formTwo.productKey,
          "tenantId": uuidv1(),
          "time": Date.parse(new Date()),
          "topic": this.formTwo.topic,
          "userId": this.formTwo.userId,
          "userType":"1"
      }
      console.info(JSON.stringify(sendData));
      var options = {
        qos :1
      }
      this.publishClient.publish(this.formTwo.topic, JSON.stringify(sendData), options);
      this.$Message.info('发布成功');
      this.publishLog += "\n";
      this.publishLog += JSON.stringify(sendData);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.home {
  display: flex;
  flex-direction: row;
}

.left {
  flex: 1;
  padding: 20px;
}

.right {
  flex: 1;
  padding: 20px;
}
</style>
