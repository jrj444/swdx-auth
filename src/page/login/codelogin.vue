<template>
  <el-form class="login-form"
           status-icon
           :rules="loginRules"
           ref="loginForm"
           :model="loginForm"
           label-width="0">
    <el-form-item v-if="tenantMode" prop="tenantId">
      <el-input
        @keyup.enter="handleLogin"
        v-model="loginForm.tenantId"
        auto-complete="off"
        :placeholder="$t('login.tenantId')"
      >
        <template #prefix>
          <i class="icon-quanxian" />
        </template>
      </el-input>
    </el-form-item>
    <el-form-item prop="phone">
      <el-input size="small"
                @keyup.enter.native="handleLogin"
                v-model="loginForm.phone"
                auto-complete="off"
                :placeholder="$t('login.phone')">
        <i slot="prefix"
           class="icon-shouji"/>
      </el-input>
    </el-form-item>
    <el-form-item prop="code">
      <el-row :span="24">
        <el-col :span="16">
          <el-input
            @keyup.enter="handleLogin"
            v-model="loginForm.codeValue"
            auto-complete="off"
            :placeholder="$t('login.code')"
          >
            <template #prefix>
              <i class="icon-yanzhengma" />
            </template>
          </el-input>
        </el-col>
        <el-col :span="8">
          <div class="login-code">
            <div @click="handleSend" class="login-code-phone">
              {{ msgText }}
            </div>
          </div>
        </el-col>
      </el-row>
    </el-form-item>
    <el-form-item>
      <el-button size="small"
                 type="primary"
                 @click.native.prevent="handleLogin"
                 class="btn-submit">{{$t('login.submit')}}</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
import { isvalidatemobile } from "@/util/validate";
import { mapGetters } from "vuex";
import {info} from "@/api/system/tenant";
import {sendSms} from "@/api/user";
import {getTopUrl} from "@/util/util";
import {encrypt} from "@/util/sm2";
export default {
  name: "codelogin",
  data() {
    const validatePhone = (rule, value, callback) => {
      if (isvalidatemobile(value)[0]) {
        callback(new Error(isvalidatemobile(value)[1]));
      } else {
        callback();
      }
    };
    return {
      tenantMode: this.website.tenantMode,
      msgText: '',
      msgTime: '',
      msgKey: false,
      loginForm: {
        tenantId: '000000',
        phone: '',
        codeValue: '',
        codeId: '',
      },
      loginRules: {
        phone: [{ required: true, trigger: 'blur', validator: validatePhone }],
        codeValue: [{ required: true, trigger: 'blur' }],
      },
    };
  },
  created() {
    this.getTenant();
    this.getMsg();
  },
  mounted() {},
  computed: {
    ...mapGetters(["tagWel"]),
    config() {
      return {
        MSGINIT: this.$t("login.msgText"),
        MSGSCUCCESS: this.$t("login.msgSuccess"),
        MSGTIME: 60
      };
    }
  },
  props: [],
  methods: {
    handleSend() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          if (this.msgKey) return;
          this.msgText = this.msgTime + this.config.MSGSCUCCESS;
          this.msgKey = true;
          const time = setInterval(() => {
            this.msgTime--;
            this.msgText = this.msgTime + this.config.MSGSCUCCESS;
            if (this.msgTime === 0) {
              this.msgTime = this.config.MSGTIME;
              this.msgText = this.config.MSGINIT;
              this.msgKey = false;
              clearInterval(time);
            }
          }, 1000);
          sendSms(this.loginForm.tenantId, encrypt(this.loginForm.phone)).then(res => {
            const data = res.data;
            if (data.success) {
              this.loginForm.codeId = data.data.id;
              this.$message.success(data.msg);
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          const loading = this.$loading({
            lock: true,
            text: '登录中,请稍后',
            background: 'rgba(0, 0, 0, 0.7)',
          });
          this.$store
            .dispatch('LoginByPhone', this.loginForm)
            .then(() => {
              loading.close();
              this.$router.push({path: this.tagWel.value});
            })
            .catch(err => {
              console.log(err);
              loading.close();
            });
        }
      });
    },
    getMsg() {
      this.msgText = this.config.MSGINIT;
      this.msgTime = this.config.MSGTIME;
    },
    getTenant() {
      let domain = getTopUrl();
      // 临时指定域名，方便测试
      //domain = "https://bladex.cn";
      info(domain).then(res => {
        const data = res.data;
        if (data.success && data.data.tenantId) {
          this.tenantMode = false;
          this.loginForm.tenantId = data.data.tenantId;
          this.$parent.$refs.login.style.backgroundImage = `url(${data.data.backgroundUrl})`;
        }
      });
    },
  }
};
</script>

<style>
.msg-text {
  display: block;
  width: 60px;
  font-size: 12px;
  text-align: center;
  cursor: pointer;
}
.msg-text.display {
  color: #ccc;
}
</style>
