<template>
  <div class="app-container">
    <el-card class="filter-container" shadow="never">
      <div>
        <i class="el-icon-search"></i>
        <span>筛选搜索</span>
        <el-button style="float: right" @click="handleSearchList()" type="primary" size="small">查询结果</el-button>
        <el-button
          style="float: right;margin-right: 15px"
          @click="handleResetSearch()"
          size="small"
        >重置</el-button>
      </div>
      <div style="margin-top: 15px">
        <el-form :inline="true" :model="listQuery" size="small" label-width="140px">
          <el-form-item label="输入搜索：">
            <el-input style="width: 203px" v-model="listQuery.keyword" placeholder="商品名称"></el-input>
          </el-form-item>
          <el-form-item label="商品货号：">
            <el-input style="width: 203px" v-model="listQuery.productSn" placeholder="商品货号"></el-input>
          </el-form-item>
          <el-form-item label="商品分类：">
            <el-cascader clearable v-model="selectProductCateValue" :options="productCateOptions"></el-cascader>
          </el-form-item>
          <el-form-item label="商品品牌：">
            <el-select v-model="listQuery.brandId" placeholder="请选择品牌" clearable>
              <el-option
                v-for="item in brandOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="上架状态：">
            <el-select v-model="listQuery.publishStatus" placeholder="全部" clearable>
              <el-option
                v-for="item in publishStatusOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="审核状态：">
            <el-select v-model="listQuery.verifyStatus" placeholder="全部" clearable>
              <el-option
                v-for="item in verifyStatusOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
    </el-card>
    <el-card class="operate-container" shadow="never">
      <i class="el-icon-tickets"></i>
      <span>数据列表</span>
      <el-button type="primary" size="mini" style="float: right" @click="onAddPro">添加</el-button>

      <el-dialog :title="form.name ? '编辑商品' : '添加商品'" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="商品名称" :label-width="formLabelWidth">
            <el-input v-model="form.name" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="商品库存" :label-width="formLabelWidth">
            <el-input v-model="form.stock" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="商品单价" :label-width="formLabelWidth">
            <el-input v-model="form.price" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="商品图片">
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
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="addProduct">确 定</el-button>
        </div>
      </el-dialog>
    </el-card>
    <div class="table-container">
      <el-table
        ref="productTable"
        :data="list"
        style="width: 100%"
        @selection-change="handleSelectionChange"
        v-loading="listLoading"
        border
      >
        <el-table-column type="selection" width="60" align="center"></el-table-column>
        <el-table-column label="编号" width="100" align="center">
          <template slot-scope="scope">{{scope.row.id}}</template>
        </el-table-column>
        <el-table-column label="商品图片" width="120" align="center">
          <template slot-scope="scope">
            <img style="height: 80px" :src="scope.row.image" />
          </template>
        </el-table-column>
        <el-table-column label="商品名称" align="center">
          <template slot-scope="scope">
            <p>{{scope.row.name}}</p>
            <p>品牌：{{scope.row.brandName}}</p>
          </template>
        </el-table-column>
        <el-table-column label="价格/货号" width="120" align="center">
          <template slot-scope="scope">
            <p>价格：￥{{scope.row.price}}</p>
            <p>货号：{{scope.row.sn}}</p>
          </template>
        </el-table-column>
        <el-table-column label="标签" width="140" align="center">
          <template slot-scope="scope">
            <p>
              上架：
              <el-switch
                @change="handlePublishStatusChange(scope.$index, scope.row)"
                :active-value="1"
                :inactive-value="0"
                v-model="scope.row.isMarketable"
              ></el-switch>
            </p>
            <p>
              新品：
              <el-switch
                @change="handleNewStatusChange(scope.$index, scope.row)"
                :active-value="1"
                :inactive-value="0"
                v-model="scope.row.newStatus"
              ></el-switch>
            </p>
            <p>
              人气：
              <el-switch
                @change="handleRecommendStatusChange(scope.$index, scope.row)"
                :active-value="1"
                :inactive-value="0"
                v-model="scope.row.peopleStatus"
              ></el-switch>
            </p>
          </template>
        </el-table-column>
        <el-table-column label="库存" width="100" align="center">
          <template slot-scope="scope">
            {{scope.row.stock}}
          </template>
        </el-table-column>
        <el-table-column label="销量" width="100" align="center">
          <template slot-scope="scope">{{scope.row.saleNum}}</template>
        </el-table-column>
        <el-table-column label="审核状态" width="100" align="center">
          <template slot-scope="scope">
            <p>{{scope.row.status | verifyStatusFilter}}</p>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="160" align="center">
          <template slot-scope="scope">
            <p>
              <el-button size="mini" @click="handleUpdateProduct(scope.$index, scope.row)">编辑</el-button>
            </p>
            <p>
              <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
            </p>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="batch-operate-container">
      <el-select size="small" v-model="operateType" placeholder="批量操作">
        <el-option
          v-for="item in operates"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
      <el-button
        style="margin-left: 20px"
        class="search-button"
        @click="handleBatchOperate()"
        type="primary"
        size="small"
      >确定</el-button>
    </div>
    <div class="pagination-container">
      <el-pagination
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        layout="total, sizes,prev, pager, next,jumper"
        :page-size="listQuery.pageSize"
        :page-sizes="[5,10,15]"
        :current-page.sync="listQuery.pageNum"
        :total="total"
      ></el-pagination>
    </div>
    <el-dialog title="编辑货品信息" :visible.sync="editSkuInfo.dialogVisible" width="40%">
      <span>商品货号：</span>
      <span>{{editSkuInfo.productSn}}</span>
      <el-input
        placeholder="按sku编号搜索"
        v-model="editSkuInfo.keyword"
        size="small"
        style="width: 50%;margin-left: 20px"
      >
        <el-button slot="append" icon="el-icon-search" @click="handleSearchEditSku"></el-button>
      </el-input>
      <el-table style="width: 100%;margin-top: 20px" :data="editSkuInfo.stockList" border>
        <el-table-column label="SKU编号" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.skuCode"></el-input>
          </template>
        </el-table-column>
        <el-table-column
          v-for="(item,index) in editSkuInfo.productAttr"
          :label="item.name"
          :key="item.id"
          align="center"
        >
          <template slot-scope="scope">{{getProductSkuSp(scope.row,index)}}</template>
        </el-table-column>
        <el-table-column label="销售价格" width="80" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.price"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="商品库存" width="80" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.stock"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="库存预警值" width="100" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.lowStock"></el-input>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editSkuInfo.dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleEditSkuConfirm">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
// import {
//   fetchList,
//   updateDeleteStatus,
//   updateNewStatus,
//   updateRecommendStatus,
//   updatePublishStatus
// } from '@/api/product'
// import {fetchList as fetchSkuStockList,update as updateSkuStockList} from '@/api/skuStock'
// import {fetchList as fetchProductAttrList} from '@/api/productAttr'
// import {fetchList as fetchBrandList} from '@/api/brand'
// import {fetchListWithChildren} from '@/api/productCate'

const defaultListQuery = {
  keyword: null,
  pageNum: 1,
  pageSize: 5,
  publishStatus: null,
  verifyStatus: null,
  productSn: null,
  productCategoryId: null,
  brandId: null
};
export default {
  name: "productList",
  data() {
    return {
      editSkuInfo: {
        dialogVisible: false,
        productId: null,
        productSn: "",
        productAttributeCategoryId: null,
        stockList: [],
        productAttr: [],
        keyword: null
      },
      operates: [
        {
          label: "商品上架",
          value: "publishOn"
        },
        {
          label: "商品下架",
          value: "publishOff"
        },
        {
          label: "设为人气推荐",
          value: "recommendOn"
        },
        {
          label: "取消人气推荐",
          value: "recommendOff"
        },
        {
          label: "设为新品",
          value: "newOn"
        },
        {
          label: "取消新品",
          value: "newOff"
        },
        {
          label: "移入回收站",
          value: "recycle"
        }
      ],
      operateType: null,
      listQuery: Object.assign({}, defaultListQuery),
      list: null,
      total: null,
      listLoading: true,
      selectProductCateValue: null,
      multipleSelection: [],
      productCateOptions: [],
      brandOptions: [],
      publishStatusOptions: [
        {
          value: "1",
          label: "上架"
        },
        {
          value: "0",
          label: "下架"
        }
      ],
      verifyStatusOptions: [
        {
          value: "1",
          label: "审核通过"
        },
        {
          value: "0",
          label: "未审核"
        }
      ],
      dialogTableVisible: false,
      dialogFormVisible: false,
      form: {},
      formLabelWidth: "120px",
      imageUrl: ""
    };
  },
  created() {
    this.getList();
    this.getBrandList();
    this.getProductCateList();
  },
  mounted() {},
  watch: {
    selectProductCateValue: function(newValue) {
      if (newValue != null && newValue.length == 3) {
        this.listQuery.productCategoryId = newValue[1];
      } else {
        this.listQuery.productCategoryId = null;
      }
    }
  },
  filters: {
    verifyStatusFilter(value) {
      if (value === "1") {
        return "审核通过";
      } else {
        return "未审核";
      }
    }
  },
  methods: {
    getProductSkuSp(row, index) {
      let spData = JSON.parse(row.spData);
      if (spData != null && index < spData.length) {
        return spData[index].value;
      } else {
        return null;
      }
    },
    getList() {
      this.listLoading = true;
      this.$http({
        url: this.$http.adornUrl("/spu/list"),
        method: "post",
        data: this.$http.adornData(this.listQuery)
      }).then(({ data }) => {
        if (data.status === 1) {
          this.list = data.data.list;
          this.total = data.data.total;
          this.totalPage = data.data.totalPage;
          this.pageSize = data.data.pageSize;
          this.listLoading = false;
        }
      });
    },
    getBrandList() {
      this.listLoading = true;
      this.$http({
        url: this.$http.adornUrl("/brand/list"),
        method: "post",
        data: this.$http.adornData(this.listQuery)
      }).then(({ data }) => {
        if (data.status === 1) {
          let brandList = data.data.list;
          for (let i = 0; i < brandList.length; i++) {
            this.brandOptions.push({
              label: brandList[i].name,
              value: brandList[i].id
            });
          }
          this.listLoading = false;
        }
      });
    },
    getProductCateList() {
      this.$http({
        url: this.$http.adornUrl("/category"),
        method: "get",
        data: this.$http.adornData()
      }).then(({ data }) => {
        let list = data.data.list;
        this.productCateOptions = [];
        for (let i = 0; i < list.length; i++) {
          let children = [];
          let newlist = [];
          if (list[i].children != null && list[i].children.length > 0) {
            for (let j = 0; j < list[i].children.length; j++) {
              children.push({
                label: list[i].children[j].name,
                value: list[i].children[j].id
              });
              let nextChildren = [];
              if (
                list[i].children[j].children != null &&
                list[i].children[j].children.length > 0
              ) {
                for (let k = 0; k < list[i].children[j].children.length; k++) {
                  nextChildren.push({
                    label: list[i].children[j].children[k].name,
                    value: list[i].children[j].children[k].id
                  });
                }
              }
              newlist.push({
                label: list[i].children[j].name,
                value: list[i].children[j].id,
                children: nextChildren
              });
            }
          }
          this.productCateOptions.push({
            label: list[i].name,
            value: list[i].id,
            children: newlist
          });
        }
      });
    },
    handleShowSkuEditDialog(index, row) {
      this.editSkuInfo.dialogVisible = true;
      this.editSkuInfo.productId = row.id;
      this.editSkuInfo.productSn = row.sn;
      this.editSkuInfo.productAttributeCategoryId =
        row.productAttributeCategoryId;
      this.editSkuInfo.keyword = null;
      fetchSkuStockList(row.id, {
        keyword: this.editSkuInfo.keyword
      }).then(response => {
        this.editSkuInfo.stockList = response.data;
      });
      if (row.productAttributeCategoryId != null) {
        fetchProductAttrList(row.productAttributeCategoryId, {
          type: 0
        }).then(response => {
          this.editSkuInfo.productAttr = response.data.list;
        });
      }
    },
    handleSearchEditSku() {
      fetchSkuStockList(this.editSkuInfo.productId, {
        keyword: this.editSkuInfo.keyword
      }).then(response => {
        this.editSkuInfo.stockList = response.data;
      });
    },
    handleEditSkuConfirm() {
      if (
        this.editSkuInfo.stockList == null ||
        this.editSkuInfo.stockList.length <= 0
      ) {
        this.$message({
          message: "暂无sku信息",
          type: "warning",
          duration: 1000
        });
        return;
      }
      this.$confirm("是否要进行修改", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        updateSkuStockList(
          this.editSkuInfo.productId,
          this.editSkuInfo.stockList
        ).then(response => {
          this.$message({
            message: "修改成功",
            type: "success",
            duration: 1000
          });
          this.editSkuInfo.dialogVisible = false;
        });
      });
    },
    handleSearchList() {
      this.listQuery.pageNum = 1;
      this.getList();
    },
    handleAddProduct() {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$message({
            type: "success",
            message: "删除成功!"
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    handleBatchOperate() {
      if (this.operateType == null) {
        this.$message({
          message: "请选择操作类型",
          type: "warning",
          duration: 1000
        });
        return;
      }
      if (this.multipleSelection == null || this.multipleSelection.length < 1) {
        this.$message({
          message: "请选择要操作的商品",
          type: "warning",
          duration: 1000
        });
        return;
      }
      this.$confirm("是否要进行该批量操作?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        let ids = [];
        for (let i = 0; i < this.multipleSelection.length; i++) {
          ids.push(this.multipleSelection[i].id);
        }
        switch (this.operateType) {
          case this.operates[0].value:
            this.updatePublishStatus(1, ids);
            break;
          case this.operates[1].value:
            this.updatePublishStatus(0, ids);
            break;
          case this.operates[2].value:
            this.updateRecommendStatus(1, ids);
            break;
          case this.operates[3].value:
            this.updateRecommendStatus(0, ids);
            break;
          case this.operates[4].value:
            this.updateNewStatus(1, ids);
            break;
          case this.operates[5].value:
            this.updateNewStatus(0, ids);
            break;
          case this.operates[6].value:
            this.updateDeleteStatus(1, ids);
            break;
          default:
            break;
        }
        this.getList();
      });
    },
    handleSizeChange(val) {
      this.listQuery.pageNum = 1;
      this.listQuery.pageSize = val;
      this.getList();
    },
    handleCurrentChange(val) {
      this.listQuery.pageNum = val;
      this.getList();
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handlePublishStatusChange(index, row) {
      let ids = [];
      ids.push(row.id);
      this.updatePublishStatus(row.isMarketable, ids);
    },
    handleNewStatusChange(index, row) {
      let ids = [];
      ids.push(row.id);
      this.updateNewStatus(row.newStatus, ids);
    },
    handleRecommendStatusChange(index, row) {
      let ids = [];
      ids.push(row.id);
      this.updateRecommendStatus(row.peopleStatus, ids);
    },
    handleResetSearch() {
      this.selectProductCateValue = [];
      this.listQuery = Object.assign({}, defaultListQuery);
    },
    handleDelete(index, row) {
      this.$confirm("是否要进行删除操作?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        let ids = [];
        ids.push(row.id);
        this.updateDeleteStatus(1, ids);
      });
    },
    handleUpdateProduct(index, row) {
      // let params = new URLSearchParams();
      // params.append('name',this.form.name)
      // params.append('stock',this.form.stock)
      // params.append('price',this.form.price)
      // params.append('stock',row.image)
      // params.append('id',this.form.id)
      // params.append('brandId', row.brandId)
      // params.append('category3Id',row.category3Id)
      console.log(row,111111);
      this.dialogFormVisible = true;
      
      this.form = row;
      this.imageUrl = row.image;
      
    },
    handleShowProduct(index, row) {
      console.log("handleShowProduct", row);
    },
    handleShowVerifyDetail(index, row) {
      console.log("handleShowVerifyDetail", row);
    },
    handleShowLog(index, row) {
      console.log("handleShowLog", row);
    },
    updatePublishStatus(publishStatus, ids) {
      this.listLoading = true;
      let params = new URLSearchParams();
      params.append("ids", ids);
      params.append("publishStatus", publishStatus);
      this.$http({
        url: this.$http.adornUrl("/spu/updateIsMarketable"),
        method: "post",
        params: params
      }).then(({ data }) => {
        if (data.status === 1) {
          this.listLoading = false;
        }
        this.getList();
      });
    },
    updateNewStatus(newStatus, ids) {
      this.listLoading = true;
      let params = new URLSearchParams();
      params.append("ids", ids);
      params.append("recommendStatus", newStatus);
      this.$http({
        url: this.$http.adornUrl("/smsPeopleRecommend/isNewRecommend"),
        method: "post",
        params: params
      }).then(({ data }) => {
        if (data.status === 1) {
          this.listLoading = false;
        }
        this.getList();
      });
    },
    updateRecommendStatus(recommendStatus, ids) {
      let params = new URLSearchParams();
      params.append("ids", ids);
      params.append("recommendStatus", recommendStatus);
      this.$http({
        url: this.$http.adornUrl("/smsPeopleRecommend/isPeopleRecommend"),
        method: "post",
        params: params
      }).then(({ data }) => {
        if (data.status === 1) {
          this.listLoading = false;
        }
        this.getList();
      });
    },
    updateDeleteStatus(deleteStatus, ids) {
      this.listLoading = true;
      let params = new URLSearchParams();
      params.append("ids", ids);
      params.append("deleteStatus", deleteStatus);
      this.$http({
        url: this.$http.adornUrl("/spu/deleteSpu"),
        method: "post",
        params: params
      }).then(({ data }) => {
        if (data.status === 1) {
          this.listLoading = false;
        }
        this.getList();
      });
    },
    onAddPro() {
      this.form = {};
      this.imageUrl = "";
      this.dialogFormVisible = true;
    },
    addProduct() {
      //
      console.log(this.imageUrl);
      let data = {
        name : this.form.name,
        stock : this.form.stock,
        price : this.form.price,
        stock : this.form.stock,
        image : this.imageUrl,
        id : this.form.id,
        brandId : this.form.brandId,
        category3Id : this.form.category3Id,
      };
       this.$http({
        url: this.$http.adornUrl("/spu/saveOrUpdateGoods"),
        method: "post",
        data: data
      }).then(({ data }) => {
        if (data.status === 1) {
          this.listLoading = false;
        }
        this.getList();
      this.dialogFormVisible = false;

      });
      //this.form
      //this.imageUrl
    },
    handleAvatarSuccess(res, file) {
      this.imageUrl = file.response;
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
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
}
</style>
