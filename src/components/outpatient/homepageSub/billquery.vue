<template>
  <div id="billquery">

    <el-row>
      <el-card class="el-card__body">
        <el-row type="flex" justify="space-between" align="middle">
          <el-col :span="8">
            <div style="padding-left:10px; font-weight: bold; font-size: larger">Billing</div>
          </el-col>
          <el-col :span="10">
            <el-input placeholder="Enter Patient ID" v-model="searchingPid" class="input-with-select">
              <el-button @click="tryCompletePInfo" slot="append" icon="el-icon-search"></el-button>
            </el-input>
          </el-col>
        </el-row>
      </el-card>
    </el-row>

    <el-row :gutter="10" style="margin-top: 5px;">
      <el-col :span="18">
        <!--      Table      -->
        <el-row>
          <el-card class="box-card" style="min-height: 280px; max-height: 900px;">
            <div slot="header" class="clearfix">
              <span style="font-weight: bold">Billing Items</span>
              <div style="float: right;  position:relative; top:-8px;">
                <el-select v-model="confirmType" placeholder="Select Operation Type"
                           @change="changeConfirmType()">
                  <el-option
                    v-for="item in operationList"
                    :key="item"
                    :label="item"
                    :value="item">
                  </el-option>
                </el-select>
              </div>
            </div>
            <el-table
              ref="multipleTable"
              size="small"
              :data="tableData"
              tooltip-effect="dark"
              style="width: 100%"
              @selection-change="handleSelectionChange"
              :default-sort="{prop: 'billdate', order: 'descending'}">
              <el-table-column
                type="selection"
                width="55">
              </el-table-column>
              <el-table-column prop="billid" label="ID" width="80"></el-table-column>
              <el-table-column prop="itemname" label="Item Name" width="140"></el-table-column>
              <el-table-column prop="billdate" label="Date" width="140" sortable></el-table-column>
              <el-table-column prop="totalprice" label="Price" width="140"></el-table-column>
              <el-table-column prop="feename" label="Fee Name" width="140"></el-table-column>
              <el-table-column prop="paid" label="Paid" width="140" :formatter="paidFormatter"
                               :filters="[{ text: 'Yes', value: true }, { text: 'No', value: false }]"
                               :filter-method="filterTag"
                               filter-placement="bottom-end"></el-table-column>
              <el-table-column type="expand">
                <template slot-scope="props">
                  <el-form label-position="right" class="demo-table-expand">
                    <el-form-item label="Billing Category">
                      <span>{{ props.row.billcat }}</span>
                    </el-form-item>
                    <el-form-item label="Count">
                      <span>{{ props.row.count }}</span>
                    </el-form-item>
                    <el-form-item label="Is Done">
                      <span>{{ props.row.done }}</span>
                    </el-form-item>
                    <el-form-item label="Is Printed">
                      <span>{{ props.row.print }}</span>
                    </el-form-item>
                  </el-form>
                </template>
              </el-table-column>
            </el-table>
          </el-card>
        </el-row>


      </el-col>
      <el-col :span="6">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span style="font-weight: bold">Patient Information</span>
          </div>
          <div style="padding: 15px;">
            <patient-info-line property="ID" :val="patientInfo.pid"></patient-info-line>
            <patient-info-line property="Name" :val="patientInfo.pname"></patient-info-line>
            <patient-info-line property="Gender"
                               :val="typeof patientInfo.pgender=='null'?''
                                        :patientInfo.pgender?'Male':'Female' ">

            </patient-info-line>
            <patient-info-line property="Age" :val="patientInfo.page"></patient-info-line>
            <patient-info-line property="Birth Date" :val="patientInfo.pbirth"></patient-info-line>
            <patient-info-line property="Address" :val="patientInfo.paddress"></patient-info-line>
          </div>
        </el-card>
      </el-col>
    </el-row>

  </div>
</template>

<script>
  import entry_line from '../../register/components/entry_line';
  import patient_info_line from '../../register/components/patient_info_line';

  export default {
    name: "billquery",
    data() {
      return {
        tableData: [],
        multipleSelection: [],
        totalPrice: 0,

        searchingPid: "",
        patientInfo: {
          pid: null,
          pname: null,
          page: null,
          pbirth: null,
          paddress: null,
          pgender: null,
        },

        condition: {
          billcat: "",
          billdate: "",
          billid: null,
          count: null,
          done: null,
          feecode: "",
          feename: "",
          itemcode: "",
          itemname: "",
          paid: null,
          pid: null,
          print: null,
          totalprice: null,
        },

        chargeDialogVisible: false,
        refundDialogVisible: false,
        activeName: '',

        confirmType: 'Charge',
        operationList: ["Charge", "Refund", "Query"],

        paymentMethod: "",
        paymentMethods: ["现金", "微信", "支付宝"],
        change: 0,
        receivedpayment: 0,

        receiptInfo: {
          items: [],
          paymentMethod: "",
          patientInfo: "",
          date: "",
          recid: "",
          totalPrice: "",
        }
      }
    },

    components: {
      'entry-line': entry_line,
      'patient-info-line': patient_info_line
    },

    mounted: function () {
      this.searchingPid = this.$route.query.pid;

    },

    methods: {
      tryCompletePInfo: function () {
        console.log("Complete PInfo")
        var that = this;
        this.$axios({
          url: '/api/registration/tryCompletePatientInfo',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: {id: that.searchingPid},
        })
          .then(function (response) {
            console.log("success complete pinfo");
            console.log(response.data);
            if (response.data.exists === "Yes") {
              that.patientInfo = response.data.patient;
              that.patientExists = true;
              that.$message({
                message: 'Successfully retrieved patient info',
                type: 'success'
              });
              that.getBillsByPid();
            } else {
              that.$message('Did not find patient with the ID');
            }
          })
          .catch(function (error) {
            console.log(error);
          });
      },

//---------------------Get Bills-----------------------

      getBillsByPid() {
        this.condition.pid = this.patientInfo.pid;
        if (this.confirmType === "Charge") {
          console.log("getBillsByPid->Charge")
          this.condition.paid = false;
          this.getUnpaidBills();
        } else if (this.confirmType === "Refund") {
          this.condition.done = false;
          this.getUndoneBills();
        } else if (this.confirmType === "Query") {
          this.getBills();
        }
      },

      getBills() {
        var that = this;
        this.$axios({
          url: '/api/charge/getBills',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: {condition: this.condition},
        })
          .then(function (response) {
            console.log("success get BillsByPid");
            console.log(response.data);
            that.tableData = response.data;
          })
          .catch(function (error) {
            console.log(error);
          });
      },

      getUnpaidBills() {
        var that = this;
        this.$axios({
          url: '/api/charge/getUnpaidBills',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: {condition: this.condition},
        })
          .then(function (response) {
            console.log("success get Unpaid BillsByPid");
            console.log(response.data);
            that.tableData = response.data;
          })
          .catch(function (error) {
            console.log(error);
          });
      },

      getUndoneBills() {
        var that = this;
        this.$axios({
          url: '/api/charge/getUndoneBills',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: {condition: this.condition},
        })
          .then(function (response) {
            console.log("success get Undone BillsByPid");
            console.log(response.data);
            that.tableData = response.data;
          })
          .catch(function (error) {
            console.log(error);
          });
      },

//---------------------Confirm-----------------------
      changeConfirmType() {
        console.log("changeConfirmType")
        console.log(this.confirmType)
        this.getBillsByPid()

        //清空已选

      },


      onConfirm() {
        if (this.confirmType === "Charge") {
          this.submitCharge()
          this.chargeDialogVisible = false;
        } else if (this.confirmType === "Refund") {
          this.submitRefund();
          this.refundDialogVisible = false;
        }
        this.logReceipt();
        this.activeName = '';
        this.$message({
          message: 'Successfully achieved!',
          type: 'success'
        });

      },

      submitCharge() {
        console.log("submitCharge")
        var that = this;
        this.$axios({
          url: '/api/charge/changeStatesToPaid',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: that.multipleSelection,
        })
          .then(function (response) {
            console.log("success changeStatesToPaid");
            console.log(response.data);
            that.getBillsByPid();
            console.log("Refreshed")
          })
          .catch(function (error) {
            console.log(error);
          });
      },

      submitRefund() {
        console.log("submitRefund")
        var that = this;
        this.$axios({
          url: '/api/charge/refundBill',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: that.multipleSelection,
        })
          .then(function (response) {
            console.log("success deleted all undone");
            console.log(response.data);
            that.getBillsByPid();
            console.log("Refreshed")
          })
          .catch(function (error) {
            console.log(error);
          });
      },
//---------------------Preview Receipt----------------
      logReceipt() {
        var that = this;
        this.$axios({
          url: '/api/charge/logReceipt',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: {
            patientInfo: that.patientInfo,
            bills: that.multipleSelection,
            confirmType: "Refund",
            //TODO: Add chargerID 管理员here
            chargerid: 1
          }
        })
          .then(function (response) {
            console.log("success Refund->logReceipt");
            that.receiptInfo.recid = response.data.recid;
            console.log(that.receiptInfo.recid);
            window.open("receipt-preview.html?" + "recid=" + that.receiptInfo.recid);
          })
          .catch(function (error) {
            console.log(error);
          });
      },

      onPreviewReceipt() {
        this.receiptInfo.items = this.multipleSelection;
        this.receiptInfo.paymentMethod = this.paymentMethod;
        this.receiptInfo.patientInfo = this.patientInfo;
        var myDate = new Date();
        this.receiptInfo.date = myDate.toLocaleDateString();
        this.receiptInfo.recid = 12534;
        this.receiptInfo.totalPrice = this.totalPrice;
        localStorage.setItem("receiptInfo", JSON.stringify(this.receiptInfo));
      },

      handleSelectionChange(val) {
        this.multipleSelection = val;
        this.activeName = '1';
        console.log("Handle Selection Change");
        this.calculateTotalPrice();
        console.log(this.multipleSelection);
      },

      blurReceived() {
        console.log("Blur Recieved")
        this.change = this.receivedpayment - this.totalPrice;
      },

      /**
       * Helper
       * */
      calculateTotalPrice() {
        var that = this;
        this.$axios({
          url: '/api/charge/getPrintedTotalPrice',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: {
            bills: that.multipleSelection,
            confirmType: that.confirmType,
          }
        })
          .then(function (response) {
            console.log("success calculateTotalPrice");
            that.totalPrice = response.data;
          })
          .catch(function (error) {
            console.log(error);
          });
      },

      paidFormatter(row) {
        if (row.paid)
          return "Yes"
        else
          return "No"
      },

      printFormatter(row) {
        if (row.print)
          return "Yes"
        else
          return "No"
      },

//---------------------筛选-----------------------
      filterTag(value, row) {
        return row.paid === value;
      },
      filterHandler(value, row, column) {
        const property = column['property'];
        return row[property] === value;
      }
    }
  }
</script>

<style scoped>
  a {
    text-decoration: none;
    color: white;
  }

  body {
    font-family: Helvetica, sans-serif;
  }

  .el-card__body {
    padding: 5px;
  }

  .el-tag {
    background-color: #ecf5ff;
    border-color: #d9ecff;
    display: inline-block;
    padding: 0 10px;
    line-height: 30px;
    height: auto;
    font-size: 12px;
    color: #409EFF;
    border-width: 1px;
    border-style: solid;
    border-radius: 4px;
    box-sizing: border-box;
    white-space: pre-line;
  }

  .el-table .used-row {
    background: #8492a6;
  }

  .el-collapse-item__header {

    margin-top: 0;
    font-size: medium;
    font-weight: bold;
    margin-left: 20px;
  }

</style>
