<template>
  <div id="cancel">

    <el-row>
      <el-card class="el-card__body">
        <el-row type="flex" justify="space-between" align="middle">
          <el-col :span="4">
            <div style="padding-left:10px; font-weight: bold; font-size: larger">Receipt Management</div>
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
              <span style="font-weight: bold">Receipt History</span>
              <div style="float: right;  position:relative; top:-8px;">
                <el-button style="margin-left: 20px" size="medium" icon="el-icon-refresh"
                           @click="getReceipts"
                           type="success" plain round>Reload
                </el-button>
              </div>
            </div>
            <el-table
              ref="multipleTable"
              size="small"
              :data="tableData"
              tooltip-effect="dark"
              style="width: 100%"
              :default-sort = "{prop: 'billdate', order: 'descending'}">
              <el-table-column
                type="selection"
                width="55">
              </el-table-column>
              <el-table-column prop="recid" label="ID" width="140"></el-table-column>
              <el-table-column prop="recstate" label="State" width="200"></el-table-column>
              <el-table-column prop="date" label="Date Created" width="240" sortable></el-table-column>
              <el-table-column prop="totalprice" label="Price" width="200"></el-table-column>
              <el-table-column fixed="right" label="Operations" width="120">
                <template slot-scope="scope">
                  <el-button @click="onReprint(scope.row)" type="text" size="small">
                    重打
                  </el-button>
                  <el-button @click="onReprint(scope.row)" type="text" size="small">
                    补打
                  </el-button>
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
            <patient-info-line property="Gender" :val="patientInfo.pgender"></patient-info-line>
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
  import entry_line from './components/entry_line';
  import patient_info_line from './components/patient_info_line';
  export default {
    name: "receipt",
    components:{
      'entry-line': entry_line,
      'patient-info-line': patient_info_line
    },

    data(){
        return {
          tableData: [
            {}
          ],
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

          condition : {
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
          operationList:["补打","重打"],

          paymentMethod:"",
          paymentMethods:["现金","微信","支付宝"],
          change: 0,
          receivedpayment: 0,

          receiptInfo : {
            items : [],
            paymentMethod: "",
            patientInfo: "",
            date: "",
            recid: "",
            totalPrice:"",
            recid: ""
          }
        }
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
            if (response.data.exists == "Yes") {
              that.patientInfo = response.data.patient;
              that.patientExists = true;
              that.$message({
                message: 'Successfully retrieved patient info',
                type: 'success'
              });
              that.getReceipts();
            } else {
              that.$message('Did not find patient with the ID');
            }
          })
          .catch(function (error) {
            console.log(error);
          });
      },

//---------------------Get Receipts-----------------------

      getReceipts(){
        console.log("getReceipts")
        var that = this;
        this.$axios({
          url: '/api/charge/getReceipts',
          method: 'post',
          contentType: 'application/json', // 这句不加出现415错误:Unsupported Media Type
          data: {
            pid: that.patientInfo.pid
          },
        })
          .then(function (response) {
            console.log("success get receipts");
            console.log(response.data);
            that.tableData = response.data;
          })
          .catch(function (error) {
            console.log(error);
          });
      },

      onReprint(row){
        // window.open("receipt-preview?"+"recid="+row.recid);
        console.log("here")
        this.$router.push({ path: '/receipt_preview/'+ row.recid })
      }
    }
  }
</script>

<style>
  a{
    text-decoration:none;
    color: white;
  }
  body{
    font-family: Helvetica, sans-serif;
  }
  .el-card__body{
    padding: 5px;
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
