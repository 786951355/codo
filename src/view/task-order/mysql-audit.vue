<template>
  <div style="height:100%; background:#eee">
    <Card style="height:100%; ">
      <Row style="margin-top:30px;">
        <Col
          span="16"
          offset="1"
        >
        <Form
          ref="formValidate"
          :model="formValidate"
          :rules="ruleValidate"
          :label-width="80"
        >
          <FormItem
            label="选择应用"
            prop="publish_name"
          >
            <Select
              v-model="formValidate.publish_name"
              @on-change="handleSelect(formValidate.publish_name)"
              placeholder="${publish_name} Select your publish app name"
            >
              <Option
                v-for="(name, index) in publishAppList"
                :value="name"
                :key="`${_uid}_${index}`"
              >{{name}}</Option>
            </Select>
          </FormItem>
          <FormItem
            label="选择模板"
            prop="temp_id"
          >
            <Select
              v-model="formValidate.temp_id"
              placeholder="${temp_id} 请选择你要审核执行的数据库对应的区域任务模板"
            >
              <Option
                v-for="(item, index) in sqlTempList"
                :value="item.temp_id"
                :key="`${_uid}_${index}`"
              >{{item.temp_name}}</Option>
            </Select>
          </FormItem>
          <FormItem
            label="SQL文件地址"
            prop="db_file"
          >
            <Input
              v-model="formValidate.db_file"
              placeholder="请输入你的数据语句的地址，当前只接受git库里的文件"
            ></Input>
          </FormItem>
          <FormItem
            label="开始时间"
            prop="start_time"
          >
            <DatePicker
              v-model="formValidate.start_time"
              type="datetime"
              :options="optionsDate"
              format="yyyy-MM-dd HH:mm:ss"
              show-week-numbers
              editable
              placeholder="选择一个执行时间，用来触发模板中的定时触发器"
              style="width: 330px"
            ></DatePicker>
          </FormItem>
          <FormItem>
            <Button
              type="primary"
              :loading="btn_loading"
              @click="handleSubmit('formValidate')"
            >提交</Button>
            <Button
              @click="handleReset('formValidate')"
              style="margin-left: 8px"
            >重置</Button>
          </FormItem>
        </Form>
        </Col>
        <Col
          span="4"
          offset="1"
        >
        <Alert show-icon>
          <h4 style="color: #ed4014">
            <p>1.当前页面使用应用关联数据库，去获取数据库信息，所以应用配置需关联相关的数据库.</p>
            <p>2.本服务多部署在内网，为了适配多云多区域，需要经过跳板机来链接远端数据库，请用强制主机来制定当前区域的执行主机，并在此主机部署相关服务，具体请参考部署文档</p>
            <p>3.请自行创建SQL审核的模板，并以SQL审核开头命名，例如：SQL审核-内网，SQL审核-阿里云华东一</p>
            <p>4.请把要执行的数据库语句以文本格式放在git上，任务会通过地址进行下载</p>
          </h4>
        </Alert>
        </Col>
      </Row>
    </Card>
  </div>
</template>

<script>
import {
  getPublishApplist,
  operationMysqlAudit,
  getTemplist
} from "@/api/task";
export default {
  name: "",
  data() {
    return {
      btn_loading: false,
      publishAppList: [],
      sqlTempList: [],
      publishInfo: {},
      formValidate: {
        publish_name: "",
        temp_id: "",
        db_file: ""
      },
      optionsDate: {
        disabledDate(date) {
          return date && date.valueOf() < Date.now() - 86400000;
        }
      },
      ruleValidate: {
        publish_name: [
          {
            required: true,
            message: "The app cannot be empty",
            trigger: "change"
          }
        ],
        temp_id: [
          {
            required: true,
            message: "The task template cannot be empty",
            trigger: "blur"
          }
        ],
        db_file: [
          {
            required: true,
            message: "The databases file cannot be empty",
            trigger: "blur"
          }
        ],
        start_time: [
          {
            required: true,
            type: "date",
            message: "Please select the date",
            trigger: "change"
          }
        ]
      }
    };
  },
  methods: {
    // 获取当前用户发布配置信息
    getPublishAppList() {
      getPublishApplist().then(res => {
        if (res.data.code === 0) {
          this.publishAppList = res.data.data;
        } else {
          this.$Message.error(`${res.data.msg}`);
        }
      });
    },
    handleSelect(value) {
      getPublishApplist(value).then(res => {
        if (res.data.code === 0) {
          this.publishInfo = res.data.data;
        } else {
          this.$Message.error(`${res.data.msg}`);
        }
      });
    },
    //
    getSqlTempList() {
      getTemplist().then(res => {
        if (res.data.code === 0) {
          this.sqlTempList = res.data.data.filter(
            res => res.temp_name.search("SQL审核") === 0
          );
        } else {
          this.$Message.error(`${res.data.msg}`);
        }
      });
    },
    handleClose(event, value) {
      this.$Message.error(`暂不提供删除功能`);
    },
    handleSubmit(value) {
      this.btn_loading = true;
      setTimeout(() => {
        operationMysqlAudit(this.formValidate, "post").then(res => {
          if (res.data.code === 0) {
            this.$Message.success(`${res.data.msg}`);
          } else {
            this.$Message.error(`${res.data.msg}`);
          }
        });
        this.btn_loading = false;
      }, 1000);
    },
    handleReset(value) {
      this.$refs[value].resetFields();
    }
  },
  mounted() {
    this.getPublishAppList();
    this.getSqlTempList();
  }
};
</script>

<style scoped>
</style>
