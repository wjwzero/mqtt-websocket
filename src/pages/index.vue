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
          <FormItem prop="userName">
            <Input type="text" v-model="formOne.userName" placeholder="用户名"></Input>
          </FormItem>
          <FormItem prop="password">
            <Input type="text" v-model="formOne.password" placeholder="密码"></Input>
          </FormItem>
          <FormItem>
            <Button type="primary" @click="handleSubmit('formOne')">Signin</Button>
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
            <Button type="primary" @click="handleSubmit('formTwo')">Signin</Button>
          </FormItem>
        </Form>
      </div>
      <div class="right">
        <Form ref="formThree" :model="formThree">
          <Divider>连接mqttborker</Divider>
          <FormItem prop="serviceUri">
            <Input type="text" v-model="formThree.serviceUri" placeholder="地址"></Input>
          </FormItem>
          <FormItem prop="userName">
            <Input type="text" v-model="formThree.userName" placeholder="用户名"></Input>
          </FormItem>
          <FormItem prop="password">
            <Input type="text" v-model="formThree.password" placeholder="密码"></Input>
          </FormItem>
          <FormItem>
            <Button type="primary" @click="handleSubmit('formThree')">Signin</Button>
          </FormItem>
        </Form>
        <Form ref="formFour" :model="formFour">
          <Divider>订阅</Divider>
          <FormItem prop="topic">
            <Input type="text" v-model="formFour.topic" placeholder="主题"></Input>
          </FormItem>
          <FormItem>
            <Button type="primary" @click="onPublish()">Signin</Button>
          </FormItem>
        </Form>
      </div>
    </div>
    <div class="home">
        <div class="left">
            <Divider>接收日志</Divider>
            <Input v-model="subscribeLog" type="textarea" placeholder="接收日志" readonly></Input>  
        </div>
        <div class="right">
            <Divider>发送日志</Divider>
            <Input v-model="publishLog" type="textarea" placeholder="发送日志" readonly></Input>    
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
      elId: "",
      publishLog: "",
      subscribeLog: "",
      client: null,
      formOne: {
        serviceUri: "",
        userName: "",
        password: "",
      },
      formTwo: {
        userId: "",
        productKey: "",
        topic: "",
        data: "",
      },
      formThree: {
        serviceUri: "",
        userName: "",
        password: "",
      },
      formFour: {
        topic: "",
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
    this.init();
    // axios.get('https://api.coindesk.com/v1/bpi/currentprice.json')
    // .then(response => (
    //     console.log(response)

    // ))
  },
  created() {
      this.elId = uuidv1() //获取随机id
  },
  methods: {
    init() {
      // 连接选项
      const options = {
        clean: false, // 保留回话
        connectTimeout: 60000, // 超时时间
        // 认证信息
        clientId: "mqtttest_"+this.elId,
        username: 'emqx_test',
        password: 'emqx_test',
      };

      // 连接字符串, 通过协议指定使用的连接方式
      // ws 未加密 WebSocket 连接
      // wss 加密 WebSocket 连接
      // mqtt 未加密 TCP 连接
      // mqtts 加密 TCP 连接
      // wxs 微信小程序连接
      // alis 支付宝小程序连接
      const connectUrl = "ws://121.37.227.84:8083/mqtt";
      this.client = mqtt.connect(connectUrl, options);
      this.client.on("reconnect", (error) => {
        console.log("正在重连:", error);
      });

      this.client.on("error", (error) => {
        console.log("连接失败:", error);
      });

      this.client.on("message", (topic, message) => {
        console.log("收到消息：", topic, message.toString());
      });
    },
    handleSubmit(name) {
      console.log(this[name]);
    },
    onPublish() {
      this.client.publish("testtopic", "testtest");
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
