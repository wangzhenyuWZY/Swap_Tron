<template>
  <div class="container">
    <div class="wtrx">
      <div class="wtrx-box">
        <div class="wtrx-box1">
          <div class="wtrx-left">
            <samp class="trx">TRX <img class="wtrx_img" src="@/assets/img/icon_arrow_right.svg" alt=""> WTRX</samp>
            <div class="trx-a"> <samp class="trx-a1">TRX{{$t('Exc.Balance')}} :</samp><samp class="trx-a2"> {{trxBalance}}</samp> </div>
            <input
              type="number"
              v-model="trxNum"
              :disabled="inputdisabled1"
              onKeypress="return (/[\d,.]/.test(String.fromCharCode(event.keyCode)))"
                   :placeholder="$t('wtrx.petao')">
            <div class="trx-b"> <samp class="trx-b1">{{$t('wtrx.yog')}} WTRX:</samp><samp class="wtrx-b2">{{trxNum?trxNum:'0'}}</samp></div>
            <div class="wtr-btn">
              <el-button class="from_botton" :loading="btnLoading1" :disabled="btnDisabled1" @click="changeWtrx">{{$t('confirm')}}</el-button>
            </div>
          </div>
          <div class="wtrx-right">
            <samp class="wtrx1">WTRX <img class="wtrx_img" src="@/assets/img/icon_arrow_right.svg" alt=""> TRX</samp>
            <div class="wtrx-a"> <samp class="wtrx-a1">WTRX {{$t('Exc.Balance')}} :</samp><samp class="wtrx-a2">{{wtrxBalance}}</samp> </div>
            <input
type="number"
v-model="wtrxNum"
:disabled="inputdisabled1"
onKeypress="return (/[\d,.]/.test(String.fromCharCode(event.keyCode)))"
                   :placeholder="$t('wtrx.petao1')">
            <div class="wtrx-b"> <samp class="wtrx-b1">{{$t('wtrx.yog')}} TRX:</samp><samp class="wtrx-b2">{{wtrxNum?wtrxNum:0}}</samp></div>
            <div class="wtr-btn ">
              <el-button class="from_botton" :loading="btnLoading2" :disabled="btnDisabled2" @click="getAllowance">{{$t('confirm')}}</el-button>
            </div>
          </div>
        </div>

        <div class="wtrx-bottom">
          <p>{{$t('wtrx.Whatis')}} WTRX?</p>
          <p>{{$t('wtrx.titn')}}
            <br> {{$t('wtrx.titn1')}}
          </p>
          <p>{{$t('wtrx.CWetTRC20')}}</p>
          <p>{{$t('wtrx.wtitn')}}</p>
        </div>
      </div>
    </div>
    <valert :isShow="showAlert" :alertType='typeName' :url="typeUrl" @close='showAlert=false' />
  </div>
</template>
<script>
import ipConfig from '../../config/ipconfig.bak'
import { approved, allowance, getConfirmedTransaction } from '../../utils/tronwebFn'
import valert from '../Pool/valret'
export default {
  data() {
    return {
      wtrxContract: null,
      trxBalance: 0,
      wtrxBalance: 0,
      approveBalance: 0,
      trxNum: null,
      wtrxNum: null,
      btnDisabled1: true,
      btnDisabled2: true,
      inputdisabled1: true,
      inputdisabled2: true,
      btnLoading1: false,
      btnLoading2: false,
      proNmae: 'Confirm',
      showAlert: false,
      typeName: 'success',
      stup: 1,
      typeUrl: ''
    }
  },
  components: {
    valert
  },
  computed: {

  },
  watch: {
    trxNum(value) {
      this.trxNum = this.inputType(this.trxNum)
      if (value != '') {
        if (value <= parseInt(this.trxBalance)) {
          this.btnDisabled1 = false
          return
        }
        this.btnDisabled1 = true
      } else {
        this.btnDisabled1 = true
      }
    },
    wtrxNum(value) {
      this.wtrxNum = this.inputType(this.wtrxNum)
      if (value != '') {
        if (value <= parseInt(this.wtrxBalance)) {
          this.btnDisabled2 = false
          return
        }
        this.btnDisabled2 = true
      } else {
        this.btnDisabled2 = true
      }
    }
  },
  created() {
    this.init()
  },
  methods: {
    init() { // 初始化tronweb
      const that = this
      this.$initTronWeb().then(function(tronWeb) {
        that.gettrx()
        that.getWtrxContract()
      })
    },
    gettrx() {
      var that = this
      window.tronWeb.trx.getAccount(window.tronWeb.defaultAddress.base58).then(function(account) {
        that.trxBalance = window.tronWeb.fromSun(account.balance)
      })
    },
    async getWtrxContract() { // 链接wtrx合约
      console.log('ipConfig.wtrxAddress===========' + ipConfig.wtrxAddress)
      this.wtrxContract = await window.tronWeb.contract().at(ipConfig.wtrxAddress)
      if (this.wtrxContract) {
        this.getWtrx()
      }
    },
    async getWtrx() { // 获取wtrxbalance
      const that = this
      try {
        const res = await that.wtrxContract['balanceOf'](window.tronWeb.defaultAddress.base58).call()
        that.wtrxBalance = window.tronWeb.fromSun(res)
        this.inputdisabled1 = false
        this.inputdisabled2 = false
      } catch (error) {
        console.log(error)
      }
    },
    async changeWtrx() { // 兑换wtrx
      const that = this
      this.loading1(1)
      try {
        const res = await that.wtrxContract['deposit']().send({
          feeLimit: 100000000,
          callValue: window.tronWeb.toSun(that.trxNum),
          tokenId: 0,
          shouldPollResponse: true
        })
        if (res) {
          this.getWtrx()
          this.gettrx()
        }
        this.loading1()
      } catch (error) {
        if (error == 'Confirmation declined by user') {
          that.$message.success('success')
        }
        this.loading1()
        console.log(error)
      }
    },
    getAllowance() { // 查询授权
      const that = this
      that.loading2(1)
      allowance(ipConfig.wtrxAddress, ipConfig.wtrxAddress).then((res) => {
        if (res) {
          that.approveBalance = parseInt(res._hex ? res._hex : res.constant_result[0], 16)
          if (that.approveBalance == 0 || window.tronWeb.toSun(this.wtrxNum) > that.approveBalance) {
            // alert('    ');
            if (that.proNmae == 'approved') {
              that.loading2(1)
              // that.showAlert = true;
              approved(ipConfig.wtrxAddress, ipConfig.wtrxAddress).then(res => {
                console.log(res)
                that.proNmae = 'Confirm'
                that.loading2(0)
              }).catch(() => {
                that.$message.error('privilege grant failed')
                that.loading2(0)
              })
            } else {
              if (that.stup == 1) {
                that.proNmae = 'approved'
                that.loading2(0)
                that.stup += 1
              } else {
                that.changeTrx()
              }
            }
          } else {
            that.changeTrx()
          }
        } else {
          that.loading2()
        }
      })
    },
    async changeTrx() { // 兑换trx
      const that = this
      const contractAddress = ipConfig.wtrxAddress
      const functionSelector = 'withdraw(uint256)'
      const options = {}
      that.loading2(1)
      const parameter = [{ type: 'uint256', value: window.tronWeb.toSun(this.wtrxNum) }]
      try {
        const transaction = await window.tronWeb.transactionBuilder.triggerSmartContract(contractAddress, functionSelector, options, parameter)
        if (!transaction.result || !transaction.result.result) {
          that.loading2(0)
          return console.error('Unknown error: ' + transaction, null, 2)
        }
        window.tronWeb.trx.sign(transaction.transaction).then(function(signedTransaction) {
          window.tronWeb.trx.sendRawTransaction(signedTransaction).then(function(res) {
            that.showAlert = true
            that.typeUrl = 'https://shasta.tronscan.org/#/transaction/' + res.txid
            getConfirmedTransaction(res.txid).then((res1) => {
              console.log(res1)
              that.$message.success(this.$t('aut'))
              if (that.stup != 1) {
                that.proNmae = 'Confirm'
              }
              that.stup = 1
              that.getWtrx()
              that.gettrx()
              that.loading2(0)
            })
          })
        })
      } catch (error) {
        that.loading2(0)
        console.log(error)
      }
    },
    loading1(n) {
      if (n) {
        this.btnLoading1 = true
        this.btnDisabled1 = true
      } else {
        this.btnLoading1 = false
        this.btnDisabled1 = false
      }
    },
    loading2(n) {
      if (n) {
        this.btnLoading2 = true
        this.btnDisabled2 = true
      } else {
        this.btnLoading2 = false
        this.btnDisabled2 = false
      }
    },
    inputType(n) {
      if (n.indexOf('.') != -1) {
        const str = n.split('.')
        if (str[1].length > 8) {
          n = str[0] + '.' + str[1].slice(0, 8)
          return n
        }
      }
      return n
    }
  },
  mounted() {
  }
}
</script>

<style lang="scss" scoped>
.wtrx {
  color: #0f1730;
}
.wtrx-box {
  height: 800px;
  /* background: #cccc; */
  padding-top: 120px;

  margin: 0 auto;
  display: flex;
  align-items: center;
  flex-direction: column;
}
.wtrx-box1 {
  // margin-top: 40px;
  display: flex;
  justify-content: center;
}

.wtrx-left {
  margin-right: 15px;
  border-radius: 20px;
  background: #ffffff;
  width: 400px;
  height: 386px;
  top: 240px;
  left: 531px;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  .trx {
    margin-top: 49px;
    height: 22px;
    font-size: 22px;
    font-family: roboto-mediumitalic;
    font-weight: 500;
    color: #0f1730;
    line-height: 22px;
  }
  .trx-a {
    margin-top: 40px;

    .trx-a1 {
      height: 18px;
      font-size: 18px;
      font-family: roboto;
      font-weight: 400;
      color: #0f1730;
      line-height: 18px;

      .trx-a2 {
        height: 18px;
        font-size: 18px;
        font-family: Roboto-Regular, Roboto;
        font-weight: 400;
        color: #0f1730;
        line-height: 18px;
      }
    }
  }
  .trx-b {
    margin-top: 48px;
    .trx-b1 {
      height: 18px;
      font-size: 18px;
      font-family: roboto;
      font-weight: 400;
      color: #0f1730;
      line-height: 18px;
      .trx-b2 {
        height: 18px;
        font-size: 18px;
        font-family: Roboto-Regular, Roboto;
        font-weight: 400;
        color: #0f1730;
        line-height: 18px;
      }
    }
  }
}

.wtrx-left input {
  box-sizing: border-box;
  padding-left: 50px;
  margin-top: 12px;
  font-size: 16px;
  font-family: roboto;
  font-weight: 400;
  color: #878b97;
  line-height: 18px;
  width: 320px;
  height: 48px;
  background: #f4f5fa;
  border-radius: 24px;
}
.wtrx-right {
  width: 400px;
  height: 386px;
  border-radius: 20px;
  background: #fff;
  margin-left: 15px;
  display: flex;
  flex-direction: column;
  align-items: center;
  .wtrx1 {
    margin-top: 49px;
    height: 22px;
    font-size: 22px;
    font-family: roboto-mediumitalic;
    font-weight: 500;
    color: #0f1730;
    line-height: 22px;
  }
  .wtrx-a {
    margin-top: 39px;

    .wtrx-a1 {
      height: 18px;
      font-size: 18px;
      font-family: roboto;
      font-weight: 400;
      color: #0f1730;
      line-height: 18px;
      .wtrx-a2 {
        height: 18px;
        font-size: 18px;
        font-family: roboto;
        font-weight: 400;
        color: #0f1730;
        line-height: 18px;
      }
    }
  }
  .wtrx-b {
    margin-top: 48px;
    .wtrx-b1 {
      height: 18px;
      font-size: 18px;
      font-family: roboto;
      font-weight: 400;
      color: #0f1730;
      line-height: 18px;
      .wtrx-b2 {
        height: 21px;
        font-size: 18px;
        font-family: Roboto-Regular, Roboto;
        font-weight: 400;
        color: #0f1730;
        line-height: 21px;
      }
    }
  }
}

.wtrx-right input {
  box-sizing: border-box;
  font-size: 16px;
  font-family: roboto;
  color: #878b97;
  line-height: 18px;
  margin-top: 12px;
  width: 320px;
  height: 48px;
  background: #f4f5fa;
  border-radius: 24px;
  padding-left: 50px;
}

.wtr-btn {
  width: 320px;
  height: 56px;
  margin: 0 auto;
  margin-top: 16px;
}

.wtrx-bottom {
  width: 860px;
  height: 300px;
}
.wtrx-bottom p:nth-child(1) {
  font-family: roboto-mediumitalic;
  font-weight: 400;
  font-size: 18px;
  margin-top: 42px;
  text-align: center;
  line-height: 21px;
  color: #0f1730;
}
.wtrx-bottom p:nth-child(2) {
  font-family: roboto;
  font-weight: 400;
  margin-top: 8px;
  font-size: 16px;
  color: #a2a5ae;
  text-align: center;
  line-height: 22px;
}
.wtrx-bottom p:nth-child(3) {
  font-family: roboto-mediumitalic;
  font-weight: 500;
  margin-top: 20px;
  font-size: 18px;
  color: #0f1730;
  text-align: center;
}
.wtrx-bottom p:nth-child(4) {
  font-family: roboto;
  font-weight: 400;
  margin-top: 8px;
  font-size: 16px;
  color: #a2a5ae;
  text-align: center;
  line-height: 22px;
}
.wtrx_img {
  vertical-align: sub;
  margin: 0 4px;
  cursor: pointer;
}
.wtrx-b2 {
  max-width: 220px;
  overflow: hidden;
  vertical-align: middle;
  display: inline-block;
  margin-top: -2px;
}

@media screen and (max-width: 750px) {
  .wtrx {
    .wtrx-box1 {
      flex-wrap: wrap;
      .wtrx-right {
        margin: 0;
        width: 9rem;
        font-size: 0.5rem;
        .wtrx-a {
          margin-top: 20px;
          .wtrx-a1 {
            font-size: 0.37rem;
          }
          .wtrx-a2 {
            font-size: 0.37rem;
          }
        }
        .trx-b {
          margin-top: 1rem;
        }
      }
      .wtrx-left {
        margin: 0;
        // width: 350;
        width: 9rem;
        margin-bottom: 20px;
        font-size: 0.5rem;
        .trx {
          // font-size: 0.45rem;
        }
        .trx-b {
          margin-top: 1rem;
        }
        .trx-a {
          margin-top: 20px;
          .trx-a1 {
            font-size: 0.37rem;
          }
          .trx-a2 {
            font-size: 0.37rem;
          }
        }
        .trx-a2 {
          font-size: 0.37rem;
        }
      }
    }
    .wtrx-box {
      height: 100%;
      padding-top: 0;
      // padding: 0 0.4rem;
    }
    .wtrx-bottom {
      width: 100%;
      font-size: 0.4rem;
      p:nth-child(2),
      p:nth-child(4) {
        font-size: 0.3rem;
        padding: 0 0.5rem;
      }
      p:nth-child(4) {
        padding-bottom: 1rem;
      }
      p {
      }
    }
  }
}
</style>
