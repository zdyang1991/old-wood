<template>
  <div class="GeneralReadingDetail">
    <div class="child_head">
      <div class="block"></div>
      <div class="child_title">{{title1}}</div>
      <div class="edit">
        <el-button size="mini" icon="el-icon-edit" @click="editTitle" circle></el-button>
      </div>
      <div class="back">
        <el-button icon="el-icon-back" size="mini" @click="back"   type="primary">返回</el-button>
      </div>
    </div>
    <div class="mainbody">
      <el-collapse v-model="activeNames">
        <el-collapse-item name="1">
          <template slot="title">
            <div class="title">
              <img src="../../assets/reading.png" alt>
              <span>全科阅读</span>
              <el-button icon="el-icon-edit" @click.stop="edit(1)">编辑</el-button>
            </div>
          </template>
          <div v-html="content" class="content">{{content}}</div>
        </el-collapse-item>
        <el-collapse-item name="2">
          <template slot="title">
            <div class="title">
              <img src="../../assets/reading.png" alt>
              <span>阅读心得</span>
            </div>
          </template>
          <div class="table">
            <el-table
              :data="data"
              border
              style="width: 100%"
              :row-style="{'color':'#333333','font-size':'16px'}"
              :header-cell-style="{'color':'#666666','font-size':'16px'}"
            >
              <el-table-column prop="className" label="班级"></el-table-column>
              <el-table-column prop="studentName" label="学生"></el-table-column>
              <el-table-column prop="groupName" label="小组"></el-table-column>
              <el-table-column prop="title" label="阅读心得"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <img
                    src="/static/view1.png"
                    onmouseover="this.src=('/static/view2.png')"
                    onmouseout="this.src=('/static/view1.png')"
                    @click="view(scope.row.title,scope.row.content)"
                  >
                </template>
              </el-table-column>
            </el-table>
          </div>
        </el-collapse-item>
      </el-collapse>
      <el-dialog
        :visible.sync="dialogVisible"
        width="80%"
        custom-class="dialog"
        :close-on-click-modal="false"
        top="7vh"
        :beforeClose="close"
      >
        <ue :content="childcontent" ref="ue" v-on:childByValue="childByValue" v-if="hackReset"></ue>
      </el-dialog>
      <el-dialog
        :visible.sync="showVisible"
        width="60%"
        :title="title"
        custom-class="dialog"
        :close-on-click-modal="false"
        top="15vh"
      >
        <div class="content2">{{content2}}</div>
      </el-dialog>
    </div>
    <div class="editTtitle">
      <el-dialog title="修改标题" :visible.sync="editvisible">
        <el-form ref="form" label-width="80px">
          <el-form-item label="标题">
            <el-input v-model="title2"></el-input>
          </el-form-item>
          <el-form-item label="简写标题">
            <el-input v-model="simpletitle"></el-input>
          </el-form-item>
          <el-form-item label="图片地址">
            <el-input v-model="url"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSubmit">保存</el-button>
            <el-button @click="editvisible=false">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>
    </div>
  </div>
</template>
<script>
import ue from "../Ue.vue";
export default {
  components: {
    ue
  },
  data() {
    return {
      data: [],
      showVisible: false,
      content2: "",
      content: "",
      process: "",
      notice: "",
      other: "",
      activeNames: ["1", "2"],
      childcontent: "",
      dialogVisible: false,
      hackReset: true,
      tag: "",
      title: "",
      title1: "",
      title2: "",
      url: "",
      simpletitle: "",
      editvisible: false
    };
  },
  created() {
    this.getRefernence();
    this.getRefernenceDetailList();
  },
  methods: {
    view(title, content) {
      this.showVisible = true;
      this.content2 = content;
      this.title = title;
    },
    editTitle() {
      this.editvisible = true;
      this.title2 = this.title1;
      this.simpletitle = this.$route.params.title;
    },
    onSubmit() {
      let body = {
        idName: "refernenceId",
        idValue: this.$route.params.id,
        picUrl: this.url,
        tableName: "tbl_school_refernence",
        title: this.title2,
        simpleTitle: this.simpletitle
      };
      this.http(this.api.setTitleByID, body).then(res => {
        if (res.data.code === "0000") {
          this.$message.success("操作成功");
          this.editvisible = false;
          this.getRefernence();
        }
      });
    },
    getRefernence() {
      let body = {
        refernenceId: this.$route.params.id
      };
      this.http(this.api.getRefernence, body).then(res => {
        console.log(res);
        if (res.data.code == "0000") {
          let data = res.data.data;
          this.content = data.content;
          this.title1 = data.title;
        }
      });
    },

    getRefernenceDetailList() {
      let body = {
        refernenceId: this.$route.params.id
      };
      this.http(this.api.getRefernenceDetailList, body).then(res => {
        console.log(res);
        if (res.data.code == "0000") {
          let data = res.data.data;
          this.data = data;
        }
      });
    },

    setFieldByID(fieldName, data) {
      let body = {
        tableName: "tbl_school_refernence",
        fieldName: fieldName,
        idName: "refernenceId",
        idValue: this.$route.params.id,
        data: data
      };
      this.http(this.api.setFieldByID, body).then(res => {
        if (res.data.code == "0000") {
          this.$message({
            type: "success",
            message: res.data.msg
          });
        }
      });
    },
    handleChange(val) {
      console.log(val);
    },
    edit(number) {
      console.log(number);
      switch (number) {
        case 1:
          this.childcontent = this.content;
          this.tag = 1;
          break;
      }

      this.dialogVisible = true;
      this.$nextTick(() => {
        this.hackReset = true; //重建组件
      });
    },
    close() {
      this.hackReset = false;
      this.dialogVisible = false;
    },
    back() {
      this.$router.push("/GeneralReading");
    },
    childByValue: function(childValue) {
      // childValue就是子组件传过来的值
      console.log(childValue);
      switch (this.tag) {
        case 1:
          this.content = childValue;
          this.setFieldByID("content", childValue);
          break;
      }
      this.hackReset = false;
      this.dialogVisible = false;
    }
  }
};
</script>
<style lang="less" scoped>
.GeneralReadingDetail {
  width: 100%;
  .mainbody {
    height: calc(100% - 51px);
    // background-color: #ffffff;
    // padding: 0 20px 30px 20px;
    overflow: auto;
    box-sizing: border-box;
    .title {
      width: 100%;
      vertical-align: middle;
      font-size: 16px;
      margin-left: 20px;
      font-weight: bold;
      color: rgba(51, 51, 51, 1);

      img {
        vertical-align: middle;
      }
      span {
        height: 100%;
        vertical-align: middle;
        display: inline-block;
      }
      .el-button {
        float: right;
        margin: 15px 40px 0 0;
        width: 83px;
        padding: 0 4px;
        height: 26px;
        font-size: 14px;
        font-family: PingFang-SC-Regular;
        font-weight: bold;
        // color: rgba(138, 138, 138, 1);
      }
     
    }
    .el-collapse-item {
      margin-bottom: 8px;
      .table {
        padding: 20px;
        img {
          margin-left: 10px;
          cursor: pointer;
        }
      }
    }
  }
}
</style>
