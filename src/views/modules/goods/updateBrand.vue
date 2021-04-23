<template>
  <el-card class="form-container" shadow="never">
    <el-form :model="brand" :rules="rules" ref="brandFrom" label-width="150px">
      <el-form-item label="品牌名称：" prop="name">
        <el-input v-model="brand.name"></el-input>
      </el-form-item>
      <el-form-item label="品牌首字母：">
        <el-input v-model="brand.letter"></el-input>
      </el-form-item>
      <el-form-item label="品牌LOGO：" prop="logo">
        <!-- <single-upload v-model="brand.logo"></single-upload> -->
        <el-upload
          class="avatar-uploader"
          action="http://360i160z91.qicp.vip/api/upload"
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          :before-upload="beforeAvatarUpload"
        >
          <img v-if="imageUrl" :src="imageUrl" class="avatar" />
          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
      </el-form-item>
      <el-form-item label="品牌故事：">
        <el-input placeholder="请输入内容" type="textarea" v-model="brand.story" :autosize="true"></el-input>
      </el-form-item>
      <el-form-item label="排序：" prop="sort">
        <el-input v-model.number="brand.seq"></el-input>
      </el-form-item>
      <el-form-item label="是否显示：">
        <el-radio-group v-model="brand.showStatus">
          <el-radio :label="1">是</el-radio>
          <el-radio :label="0">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="品牌制造商：">
        <el-radio-group v-model="brand.factoryStatus">
          <el-radio :label="1">是</el-radio>
          <el-radio :label="0">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit('brandFrom')">提交</el-button>
        <el-button v-if="!isEdit" @click="resetForm('brandFrom')">重置</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>
<script>
//   import {createBrand, getBrand, updateBrand} from '@/api/brand'
//   import SingleUpload from '@/components/Upload/singleUpload'
const defaultBrand = {
  bigPic: "",
  brandStory: "",
  factoryStatus: 0,
  firstLetter: "",
  logo: "",
  name: "",
  showStatus: 0,
  sort: 0
};
export default {
  name: "BrandDetail",
  // components:{SingleUpload},
  props: {
    isEdit: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      brand: Object.assign({}, defaultBrand),
      brandId: 0,
      rules: {
        name: [
          { required: true, message: "请输入品牌名称", trigger: "blur" },
          {
            min: 2,
            max: 140,
            message: "长度在 2 到 140 个字符",
            trigger: "blur"
          }
        ],
        // logo: [{ required: false, message: "请输入品牌logo", trigger: "blur" }],
        sort: [{ type: "number", message: "排序必须为数字" }]
      },
      imageUrl: ""
    };
  },
  created() {
    if (this.isEdit) {
      getBrand(this.$route.query.id).then(response => {
        this.brand = response.data;
      });
    } else {
      this.brand = Object.assign({}, defaultBrand);
    }
  },
  activated() {
    this.brandId = this.$route.query.id;
    this.getBrandDetails();
  },
  methods: {
    handleAvatarSuccess(res, file) {
      this.imageUrl = file.response;
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg" || "image/png";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    onSubmit(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$confirm("是否提交数据", "提示", {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          }).then(() => {
            if (this.isEdit) {
              console.log(this.brand);

              let data = {
                id: this.brandId,
                name: this.brand.name,
                brandStory: this.brand.story,
                factoryStatus: this.brand.factoryStatus,
                firstLetter: this.brand.letter,
                logo: this.imageUrl,
                showStatus: this.brand.showStatus,
                sort: this.brand.seq
              };
              this.$http({
                url: this.$http.adornUrl("/brand/createOrUpdate"),
                method: "post",
                data: data
              }).then(({ data }) => {
                if (data.status === 1) {
                  this.listLoading = false;
                }
              });
            } else {
              console.log(this.brand);
              let data = {
                id: this.brandId,
                name: this.brand.name,
                brandStory: this.brand.story,
                factoryStatus: this.brand.factoryStatus,
                firstLetter: this.brand.letter,
                logo: this.imageUrl,
                showStatus: this.brand.showStatus,
                sort: this.brand.seq
              };
              this.$http({
                url: this.$http.adornUrl("/brand/createOrUpdate"),
                method: "post",
                data: data
              }).then(({ data }) => {
                if (data.status === 1) {
                  this.listLoading = false;
                }
              });
            }
          });
        } else {
          this.$message({
            message: "验证失败",
            type: "error",
            duration: 1000
          });
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
      this.brand = Object.assign({}, defaultBrand);
    },
    getBrandDetails() {
      //获取品牌数据
      this.$http({
        url: this.$http.adornUrl("/brand/getInfo/" + this.brandId),
        method: "get"
      }).then(({ data }) => {
        if (data.status === 1) {
          this.brand = data.data;
        }
      });
    }
  }
};
</script>
<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>


