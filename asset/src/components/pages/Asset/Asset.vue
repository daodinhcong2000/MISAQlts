<template>
  <div class="content">
    <div class="filter-bar">
      <div class="filter-left">
        <input
          type="text"
          placeholder="Tìm kiếm"
          class="input-search"
          v-model="searchKeyword"
          @keyup.enter="search"
        />
        <div class="icon-search"></div>
      </div>
      <div class="filter-right">
        <button @click="AddAsset()">Thêm</button>
        <button class="m-btn-refresh" @click="refreshData()"></button>
        <button class="m-btn-delete"></button>
      </div>
    </div>
    <div class="data">
      <table
        id="tbListData"
        cellspacing="0"
        cellpadding="0"
        class="el-table__body"
        style="min-width: 100%"
      >
        <thead class="has-gutter">
          <tr class="el-table__row">
            <th style="width: 3%">STT</th>
            <th style="width: 10%">MÃ TÀI SẢN</th>
            <th style="width: 27%">TÊN TÀI SẢN</th>
            <th style="width: 17%">LOẠI TÀI SẢN</th>
            <th style="width: 23%">PHÒNG BAN</th>
            <th class="cell-number" style="width: 10%">NGUYÊN GIÁ</th>
            <th style="width: 5%; text-align: center">CHỨC NĂNG</th>
          </tr>
        </thead>
        <tbody class="show-data">
          <div v-if="!this.assets.length" class="no-data">
            Không có dữ liệu hiển thị !
          </div>
          <tr
            class="el-table__row"
            v-for="(asset, index) in assets"
            :key="index"
            multiple
          >
            <td style="width: 3%; text-align: center !important">
              {{ index + 1 }}
            </td>
            <td style="width: 10%">{{ asset.assetCode }}</td>
            <td style="width: 27%">{{ asset.assetName }}</td>
            <td style="width: 17%">
              {{ getAssetTypeName(asset.assetTypeId) }}
            </td>
            <td style="width: 23%">
              {{ getDepartmentName(asset.departmentId) }}
            </td>
            <td style="width: 10%" class="cell-number">
              {{ formatCurrency(asset.originalPrice) }}
            </td>
            <td style="width: 5%; align-item: center" class="asset-operation">
              <div
                class="icon-group"
                v-show="operation"
                style="align-item: center"
              >
                <div class="icon icon-edit" @click="showDetail(asset)"></div>
                <div
                  class="icon icon-delete"
                  @click="confirmDelete(asset, index)"
                ></div>
                <div class="icon icon-history"></div>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="footer-content">
      <div>Tổng số tài sản: {{ this.assets.length }}</div>
      <div>Tổng ngyên giá: {{ totalPrice() }}</div>
    </div>
    <AssetDetail
      v-show="currentState"
      @closeDialog="closeDialog()"
      v-bind:asset="asset"
      v-bind:departments="departments"
      v-bind:assetTypes="assetTypes"
      v-bind:errors="errors"
      @changeAsset="changeAsset()"
    />

    <Alert
      v-if="alertState"
      @aceptDelete="aceptDelete()"
      @cancleDelete="cancleDelete()"
    />
    <ContextMenu @addAsset="AddAsset()" />
    <FlashMessage />
    <Notify  v-bind:message="message" v-if="statusNotify"/>
  </div>
</template>


<script>
import AssetDetail from "./AssetDetail";
import Alert from "../../common/Alert";
import axios from "axios";
import ContextMenu from "../../common/ContextMenu";
import contextMenu from "../../../js/comon/contextMenu";
import FlashMessage from "@smartweb/vue-flash-message";
import Notify from '../../common/Notify.vue';
import { Notyf } from "notyf";
import "notyf/notyf.min.css";
const notyf = new Notyf({
  duration: 3000,
  position: { x: "right", y: "bottom" },
});
export default {
  name: "Asset",
  components: {
    AssetDetail,
    Alert,
    ContextMenu,
    FlashMessage,
    Notify,
  },
  props: {},
  data: function () {
    return {
      currentState: false,
      BASE_URL: "https://localhost:44302",
      assets: [],
      assetTypes: [],
      departments: [],
      operation: true,
      asset: {},
      searchKeyword: "",
      alertState: false,
      errors: {
        errorCode: "",
        errorName: "",
      },
      index: 0,
      message: "",
      statusNotify: false,
    };
  },
  methods: {
    //CreateBy: DDCong(26/03/2021)
    confirmDelete(asset, index) {
      this.alertState = true;
      this.asset = asset;
      this.index = index;
    },
    //CreateBy: DDCong(26/03/2021)
    cancleDelete() {
      this.alertState = false;
    },
    //CreateBy: DDCong(26/03/2021)
    aceptDelete() {
      this.deleteAsset(this.asset);
    },
    //thêm dữ liệu tài sản
    //CreateBy: DDCong(26/03/2021)
    AddAsset() {
      console.log(this.flashMessage.success({ title: 'Success Title', message: "text" }));
      this.currentState = true;
      this.asset = {};
    },
    //Xem dữ liệu tài sản
    //CreateBy: DDCong(26/03/2021)
    showDetail(asset) {
      this.asset = asset;

      this.asset.departmentName = this.departments.find(
        (department) => department.departmentId === asset.departmentId
      )?.departmentName;
      this.asset.assetTypeName = this.assetTypes.find(
        (assetType) => assetType.assetTypeId === asset.assetTypeId
      )?.assetTypeName;
      // this.$root.getAssetTypeName();
      this.asset.departmentCode = this.departments.find(
        (department) => department.departmentId === asset.departmentId
      )?.departmentCode;
      // console.log(this.departments);
      this.asset.assetTypeCode = this.assetTypes.find(
        (assetType) => assetType.assetTypeId === asset.assetTypeId
      )?.assetTypeCode;
      //  this.$root.getDepartmentName();
      this.currentState = true;
      // this.asset.originalPrice = this.formatCurrency(this.asset.originalPrice);
      this.asset = JSON.parse(JSON.stringify(asset));
    },
    //Đóng bản thêm dữ liệu
    closeDialog() {
      this.errors.errorCode = "";
      this.errors.errorName = "";
      this.currentState = false;
    },
    ///Định dạng tiền
    //CreateBy : DDCONG(25/03/2021)
    formatCurrency(number) {
      return number?.toLocaleString("de-DE", { minimumFractionDigits: 0 });
    },
    // convert Id loại tài sản => tên tài sản
    // CreatedBy: DDCong(25/03/2021)
    getAssetTypeName(assetTypeId) {
      return this.assetTypes.find(
        (assetType) => assetType.assetTypeId === assetTypeId
      )?.assetTypeName;
    },

    //Convert Id phòng ban sang tên phòng ban
    // CreatedBy: DDCong(25/03/2021)
    getDepartmentName(departmentId) {
      return this.departments.find(
        (department) => department.departmentId === departmentId
      )?.departmentName;
    },

    //Hiển thị chức năng cho từng dòng dữ liệu
    showOperation() {
      return (this.operation = true);
    },

    //Tính tổng nguyên giá
    totalPrice() {
      var sum = 0;

      this.assets.forEach((asset) => {
        sum = sum + asset.originalPrice;
      });
      return this.formatCurrency(sum);
    },

    //Tait lại dữ liệu
    refreshData() {
      this.refreshAsset();
      this.refreshAssetType();
      this.refreshDepartment();
    },
    // Tải lại tài sản
    // CreatedBy: DDCong(26/3)
    async refreshAsset() {
      try {
        this.assets = [];
        const responseAsset = await axios.get(this.BASE_URL + "/api/v1/assets");
        this.assets = responseAsset.data;
        console.log(this.assets.length);
      } catch (error) {
        console.log(error);
      }
    },

    // Tải lại loại tài sản
    // CreatedBy: DDCong(26/3)
    async refreshAssetType() {
      try {
        const responseAssetType = await axios.get(
          this.BASE_URL + "/api/v1/assetTypes"
        );
        this.assetTypes = responseAssetType.data;
      } catch (error) {
        console.log(error);
      }
    },

    // Tải lại phòng ban
    // CreatedBy: DDCONG
    async refreshDepartment() {
      try {
        const responseDepartment = await axios.get(
          this.BASE_URL + "/api/v1/departments"
        );
        this.departments = responseDepartment.data;
      } catch (error) {
        console.log(error);
      }
    },

    //Tìm kiếm dữ liệu
    async search() {
      try {
        if (this.searchKeyword == "") {
          return this.refreshData();
        } else {
          const response = await axios.get(
            this.BASE_URL + "/api/v1/assets/search",
            {
              params: {
                contentFilter: this.searchKeyword,
              },
            }
          );
          this.assets = response.data;
        }
      } catch (error) {
        console.log(error);
      }
    },

    //Thêm hoặc sửa tài sản
    async changeAsset() {
      if (this.asset.assetId == undefined) {
        const response = await axios.post(
          this.BASE_URL + "/api/v1/assets",
          this.asset
        );
        if (response.data.misaCode == 900) {
          this.errors.errorCode = response.data.messenger;
        } else {
          this.assets.push(this.asset);
          console.log(this.errors.errorCode);
          this.currentState = false;
        }
        console.log(response);
        if(response.status == 201) {
            notyf.success("Thêm thành công");
        }
        else notyf.error("Không thành công");
        return response;
      } else {
        console.log(this.asset);
        const response = axios.put(
          this.BASE_URL + "/api/v1/assets",
          this.asset
        );
        this.currentState = false;
        console.log(response);
        this.refreshAsset();
         if(response.status == 200) {
            notyf.success("Sửa thành công");
        }
        else notyf.error("Sửa thất bại");
        return response;
      }
    },

    //Xóa 1 bản ghi
    async deleteAsset(asset) {
      this.alertState = false;
      const response = await axios.delete(this.BASE_URL + "/api/v1/assets", {
        params: {
          ids: asset.assetId,
        },
      });
      // this.refreshData();
      this.assets.splice(this.index, 1);
      if(response.status == 200) {
        notyf.success("Xóa thành công");
      }
      else notyf.error("Không xóa được");
      return response;
    },
  },
  mounted() {
    contextMenu.showContextMenuWithTable();
    contextMenu.hideContextMenu();
  },

  ///Lấy dữ liệu từ database
  ///CreatBy: DDCong(25-03-2021)
  async created() {
    try {
      const responseAsset = await axios.get(this.BASE_URL + "/api/v1/assets");
      this.assets = responseAsset.data;

      const responseAssetType = await axios.get(
        this.BASE_URL + "/api/v1/assetTypes"
      );
      this.assetTypes = responseAssetType.data;

      const responseDepartment = await axios.get(
        this.BASE_URL + "/api/v1/departments"
      );
      this.departments = responseDepartment.data;
    } catch (error) {
      console.log(error);
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url("../../../css/pages/dictionary/QLTS.css");
@import url("../../../css/common/filter.css");
@import url("../../../css/common/table.css");
</style>
