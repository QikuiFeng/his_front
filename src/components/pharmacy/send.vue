<template>
  <div id="send">
    <el-row :gutter="20">
      <el-col :span="8">
        <el-input type="text" v-model="pid" placeholder="Patient ID" clearable></el-input>
      </el-col>
      <el-col :span="8">
        <el-button icon="el-icon-search" type="primary" @click="getBill">Search</el-button>
      </el-col>
      <el-col :span="8">
        <el-button icon="el-icon-sold-out" type="primary" :disabled="this.multipleSelection.length === 0"
                   @click="sendMed(multipleSelection)">Send Medicine</el-button>
      </el-col>
    </el-row>


    <el-table
      :data="send"
      style="width: 100%"
      tooltip-effect="dark"
      @selection-change="handleSelectionChange"
      ref="multipleTable">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column
        prop="itemname"
        label="Medicine Name"
        width="300">
      </el-table-column>
      <el-table-column
        prop="count"
        label="Count"
        width="150">
      </el-table-column>
      <el-table-column
        prop="realname"
        label="Doctor Name"
        width="250">
      </el-table-column>
      <el-table-column
        prop="prename"
        label="Prescription Name"
        width="250">
      </el-table-column>
      <el-table-column
        prop="billdate"
        label="Opened Date">
      </el-table-column>
    </el-table>

    <div style="margin-top: 20px">
      <el-button :disabled="this.multipleSelection.length === 0" @click="toggleSelection()">Cancel Selection</el-button>
    </div>

  </div>
</template>

<script>
    export default {
        name: "send",
      data(){
          return{
            pid:null,
            itemname: "",
            count:"",
            realname:"",
            prename:"",
            billdate:"",
            send:[],

            multipleSelection: [],
          }
      },

      methods: {
        getBill: function () {
          var that = this;
          var params = new URLSearchParams();
          params.append("pid", this.pid);
          this.$axios.post('/api/pharmacy/getBill', params)
            .then(function (response) {
              console.log(response.data);
              that.send = response.data;
            })
            .catch(function (error) {
              console.log(error);
            });
        },
        sendMed: function () {
          var ids= this.multipleSelection.map(item => item.billid).join();//获取所有选中行的id组成的字符串，以逗号分隔
          console.log(ids);
          var that= this;
          this.$confirm('Are you sure you want to send all this?', 'Note', {
            confirmButtonText: 'Confirm',
            cancelButtonText: 'Cancel',
            type: 'warning'
          }).then(() => {
            this.$axios.post("/api/pharmacy/sendMed", {ids: ids})
              .then(function (response) {
                that.toggleSelection();
                that.getBill();
              })
              .catch(function (error) {
                console.log(error);
              });
            this.$message({
              type: 'success',
              message: 'Send success!'
            });
          }).catch(() => {
            this.$message({
              type: 'info',
              message: 'Cancel!'
            });
          });
        },
        toggleSelection(rows) {
          if (rows) {
            rows.forEach(row => {
              this.$refs.multipleTable.toggleRowSelection(row);
            });
          } else {
            this.$refs.multipleTable.clearSelection();
          }
        },
        handleSelectionChange(val) {
          this.multipleSelection = val;
        },
      }
    }
</script>

<style scoped>

</style>
