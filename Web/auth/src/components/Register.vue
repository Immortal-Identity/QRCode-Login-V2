<template>
  <div class="register_box">
    <router-link to="/input">
      <div class="login_close"></div>
    </router-link>
    <div class="register_panel">
      <div class="register_title">
        <img src="../assets/img/logo.png" alt="">
        <p>IPERSONA</p>
      </div>

      <el-steps :active="active" finish-status="success" simple>
        <el-step></el-step>
        <el-step></el-step>
        <el-step></el-step>
        <el-step></el-step>
      </el-steps>

      <!--      Step 1: Identity-->
      <div v-if="active===0">
        <label style="margin-top: 50px">Identity：</label>
        <input v-model="userId" type="tel" pattern="^\d{11}$" title="Please Enter Identity" required>
      </div>
      <!--      Step 2 : Email-->
      <div v-if="active===1">
        <label style="margin-top: 50px">Email：</label>
        <input v-model="userEmail" type="tel" pattern="^\d{11}$" title="Please Enter Email" required>
      </div>
      <!--      Step 3 : Password-->
      <div v-if="active===2">
        <label>Password：</label>
        <div class="input_container">
          <ul>
            <li v-bind:class="{ is_valid: contains_eight_characters }">8 Characters</li>
            <li v-bind:class="{ is_valid: contains_uppercase }">Contains Uppercase & Lowercase</li>
            <li v-bind:class="{ is_valid: contains_number }">Contains Number</li>
            <li v-bind:class="{ is_valid: contains_special_character }">Contains Special Character</li>
          </ul>
          <div class="checkmark_container" v-bind:class="{ show_checkmark: valid_password }">
            <svg width="50%" height="50%" viewBox="0 0 140 100">
              <path class="checkmark" v-bind:class="{ checked: valid_password }" d="M10,50 l25,40 l95,-70" />
            </svg>
          </div>
          <password v-model="userPassword" type="password" @input="checkPassword" class="password_first_input" title="Please Enter Password" autocomplete="off" :secureLength="secureLength" required></password>
        </div>
      </div>

      <div v-if="active===3">
        <label>Confirm Password：</label>
        <input v-model="passwordConfirm" type="password" title="Please Confirm Password" required>
        <input class="bt" @click="addUser" type="submit" value="Register">
      </div>

      <el-button style="margin-top: 12px" @click="next" v-if="active<3">NEXT</el-button>

    </div>
  </div>
</template>

<script>
import Password from 'vue-password-strength-meter'

export default {
  name: 'Register',
  data () {
    return {
      active: 0,
      userId: '',
      userEmail: '',
      userPassword: '',
      password_length: 0,
      contains_eight_characters: false,
      contains_number: false,
      contains_uppercase: false,
      contains_special_character: false,
      valid_password: false,
      passwordConfirm: '',
      secureLength: 8,
      userInfoApi: 'http://localhost/register',
      reg: /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
    }
  },
  components: {
    Password
  },
  methods: {
    checkPassword () {
      this.password_length = this.userPassword.length
      const format = /[$&+,:;=?@#|'<>.^*()%!-]/
      if (this.password_length >= 8) {
        this.contains_eight_characters = true
      } else {
        this.contains_eight_characters = false
      }
      this.contains_number = /\d/.test(this.userPassword)
      this.contains_uppercase = /[A-Z]/.test(this.userPassword) && /[a-z]/.test(this.userPassword)
      this.contains_special_character = format.test(this.userPassword)
      if (this.contains_eight_characters === true &&
        this.contains_special_character === true &&
        this.contains_uppercase === true &&
        this.contains_number === true) {
        this.valid_password = true
      } else {
        this.valid_password = false
      }
    },
    next () {
      if (this.active++ > 3) {
        this.active = 0
      }
    },
    addUser () {
      if (this.userId == null || this.userId === '') {
        this.$message.error('Identity is required!')
        return
      }
      if (this.userEmail == null || this.userEmail === '') {
        this.$message.error('userEmail is required!')
        return
      } else if (!this.reg.test(this.userEmail)) {
        this.$message.error('Please enter valid email address!')
        return
      }
      if (this.userPassword == null || this.userPassword === '') {
        this.$message.error('Password is required!')
        return
      }
      if (this.userPassword !== this.passwordConfirm) {
        this.$message.error('Password is NOT matched!')
        return
      }
      if (this.valid_password === false) {
        this.$message.error('Password is NOT valid!')
        return
      }
      this.$ajax({
        method: 'post',
        url: this.userInfoApi,
        data: {
          'user_id': this.userId,
          'user_password': this.userPassword,
          'user_email': this.userEmail
        }
      }).then((response) => {
        if (response.data.code === 202) {
          this.$message.error('UserName already exits! Please change another identity.')
        } else if (response.data.data.userId !== 0) {
          this.$message({
            message: 'Register Success!',
            type: 'success'
          })
        } else {
          this.$message.error('Register Failed, Please Check Account Or Password!')
        }
        console.log(response.data.data)
      }).catch((error) => {
        this.$message.error('Register Failed, Please Check Account Or Password!')
        console.log(error)
      })
    }
  }
}
</script>

<style scoped>
  /*登录框*/
  .register_box {
    z-index: 99;
    position: absolute;
    width: 380px;
    height: 540px;
    top: 50%;
    left: 50%;
    margin-left: -190px;
    margin-top: -270px;
    border-radius: 6px;
    background-color: #fff;
    box-shadow: 0 2px 10px #999;
  }

  .login_close {
    position: absolute;
    top: 0;
    right: 0;
    width: 64px;
    height: 64px;
    background: url(../assets/img/qrcode.png) no-repeat right top;
    background-size: 100% 100%;
    border-top-right-radius: 5px;
    cursor: pointer;
    z-index: 99;
  }

  /*登录*/
  .register_panel {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 270px;
    height: 540px;
    padding: 0 55px;
    transform: translate(-50%, -50%);
    /* background: #fff; */
    border-radius: 6px;
    overflow: hidden;
  }

  .register_panel .register_title {
    text-align: center;
  }

  .register_panel .register_title img {
    margin-top: 60px;
    height: 70px;
    width: 70px;
    border-radius: 50%;
    padding: 10px;
    border: 3px solid #d7e8fc;
  }

  .register_panel .register_title p {
    margin-top: 15px;
    color: #999999;
    font-size: 15px;
  }

  .register_panel label {
    display: block;
    font-size: 12px;
    line-height: 18px;
    color: #a9a8a5;
    margin-top: 10px;
  }

  .register_panel input {
    display: inline;
    height: 42px;
    padding: 0 5%;
    line-height: 42px;
    font-size: 14px;
    color: #333333;
    border-radius: 4px;
    outline: 0;
    border: 0;
    width: 90%;
    background: #f1f1f1;
  }

  /* 按钮 */
  .register_panel .bt {
    margin-top: 35px;
    width: 100%;
    color: #ffffff;
    background: #379df6;
    cursor: pointer;
  }

  .register_panel .bt:hover {
    background-color: #2f86f6;
  }

  ul {
    padding-left: 20px;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
  }

  li {
    margin-bottom: 8px;
    color: #a9a8a5;
    font-size: 12px;
    position: relative;
  }

  li:before {
    content: "";
    width: 0%; height: 2px;
    background: #2ecc71;
    position: absolute;
    left: 0; top: 50%;
    display: block;
    transition: all .6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  .input_container {
    position: relative;
    padding: 3px;
    border-radius: 6px;
    background: #FFF;
  }

  input[type="password"]:focus {
    border-color: rgba(50, 151, 211, .45);
  }

  .is_valid { color: rgba(136, 152, 170, 0.8); }
  .is_valid:before { width: 100%; }

  .checkmark_container {
    border-radius: 50%;
    position: absolute;
    top: -15px; right: -15px;
    background: #2ecc71;
    width: 50px; height: 50px;
    visibility: hidden;
    opacity: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: opacity .4s ease;
  }

  .show_checkmark {
    visibility: visible;
    opacity: 1;
  }

  .checkmark {
    width: 100%;
    height: 100%;
    fill: none;
    stroke: white;
    stroke-width: 15;
    stroke-linecap: round;
    stroke-dasharray: 180;
    stroke-dashoffset: 180;
  }

  .checked { animation: draw 0.5s ease forwards; }

  @keyframes draw {
    to { stroke-dashoffset: 0; }
  }

</style>
