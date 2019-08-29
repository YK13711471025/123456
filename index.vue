<template>
  <div class="basicInformation">
    <nav-bar></nav-bar>
    <div class="container">
      <!--tab栏-->
      <div class="tabs">
        <h4 :class="{'active':flag === 1}" @click="changeFlag(1)">账户设置</h4>
        <h4 :class="{'active':flag === 2}" @click="changeFlag(2)" v-has="'JSXT1101'">我的任务</h4>
      </div>
      <!--个人信息-->
      <div class="com_submenu" v-show="flag === 1">
        <div class="left">
          <div class="ghtx">
            <img :src="setUserInfo.avatar" alt="" v-if="setUserInfo.avatar">
            <img src="../../assets/img/default_boy.png" alt="" v-else>
            <p @click="showImg">
              <span>更换</span>
              <span>头像</span>
            </p>
          </div>
          <div class="content">
            <div class="cont_top">
              <h4>{{setUserInfo.userName}}</h4>
              <p>
                <img src="../../assets/img/login/ic_phone@.png" alt="">
                {{setUserInfo.phoneNumber}}
              </p>
              <Button @click="changePhone">
                <img src="../../assets/img/icon/icon-edit.png" alt="">
                <span>更换手机</span>
              </Button>
            </div>
            <div class="cont_bottom">
              <p v-if="curCompany.positionName && curCompany.departmentName">{{curCompany.departmentName}} -
                {{curCompany.positionName}}</p>
              <p v-if="!curCompany.positionName && curCompany.departmentName">{{curCompany.departmentName}}</p>
              <p v-if="curCompany.positionName && !curCompany.departmentName">{{curCompany.positionName}}</p>
              <p v-if="!curCompany.positionName && !curCompany.departmentName">暂无职位</p>
              <p>注册时间：{{setUserInfo.registTime}}</p>
              <p>登录密码：已设置</p>
              <Button @click="updatePwd">
                <img src="../../assets/img/icon/icon-edit.png" alt="">
                <span>修改密码</span>
              </Button>
            </div>
          </div>
        </div>
        <div class="right">
          <div v-if="place1.ip && place2.ip">
            <p>本次登录：{{place1.createTime}}
              {{place1.city}}（IP：{{place1.ip}}）</p>
            <p>上次登录：{{place2.createTime}}
              {{place2.city}}（IP：{{place2.ip}}）</p>
          </div>
          <div v-if="place1.ip && !place2.ip">
            <p>本次登录：{{place1.createTime}}
              {{place1.city}}（IP：{{place1.ip}}）</p>
          </div>
        </div>
      </div>
      <div class="joined" v-show="flag === 1">
        <h4>已加入的企业主体（{{setUserInfo.companys.length}}家）</h4>
        <ul>
          <li v-for="(com,index) in setUserInfo.companys" :key="index" @click="companyChange(com)">
            <div>
              <img src="../../assets/img/icon/ic_company.png" alt="">
              <p><span class="main_com">{{com.compName}}</span> <span class="cur"
                                                                      v-if="com.compId === companyId">当前主体</span></p>
              <!--<p>广东顺德伯渡服务外包营运管理有限公司</p>-->
            </div>
            <span>进入企业 <Icon type="ios-arrow-forward"/></span>
          </li>
        </ul>
      </div>
      <div v-show="flag === 2" v-has="'JSXT1101'">
        <my-task></my-task>
      </div>
    </div>
    <!--修改手机-->
    <Modal v-model="updatePhone" width="720" class="updatePhone" footer-hide>
      <div class="update_header">
        <h4>更换手机</h4>
      </div>
      <div class="update_cont">
        <div class="step">
          <div>
            <span class="circle active">1</span>
            <span>验证旧手机</span>
          </div>
          <span class="line" :class="{'active' : stepIndex !== 1}"></span>
          <div>
            <span class="circle" :class="{'active' : stepIndex !== 1}">2</span>
            <span>验证新手机</span>
          </div>
          <span class="line" :class="{'active' : stepIndex === 3}"></span>
          <div>
            <span class="circle" :class="{'active' : stepIndex === 3}">3</span>
            <span style="min-width:60px;text-align:center;">完成</span>
          </div>
        </div>
        <!--第一步-->
        <p class="p0" v-if="stepIndex === 1">请先验证您原来的手机号码：{{setUserInfo.phoneNumber}}</p>
        <Form ref="phoneData" :model="phoneData" :rules="rulePhoneData" v-show="stepIndex === 1">
          <FormItem prop="code">
            <Input v-model="phoneData.code" placeholder="请输入短信验证码" class="w204"/>
            <Button
              type="primary"
              class="btn_code"
              @click="sendCode(1)"
              v-if="isCodeShow"
            >发送验证码
            </Button>
            <Button class="btn_code" v-else>{{codeCount}}s</Button>
          </FormItem>
        </Form>
        <!--第二步-->
        <Form ref="phoneData2" :model="phoneData2" :rules="rulePhoneData2" v-show="stepIndex === 2">
          <FormItem prop="phone">
            <Input v-model="phoneData2.phone" :disabled="secondStep" placeholder="请输入新手机号码" class="w330"/>
          </FormItem>
          <FormItem prop="code">
            <Input v-model="phoneData2.code" placeholder="请输入短信验证码" class="w204"/>
            <Button
              type="primary"
              class="btn_code"
              @click="sendCode(2)"
              v-if="isCodeShow"
            >发送验证码
            </Button>
            <Button class="btn_code" v-else>{{codeCount}}s</Button>
          </FormItem>
        </Form>
        <!--第三步-->
        <div class="content">
          <img src="../../assets/img/login/succsee3.png" alt="" v-show="stepIndex === 3">
        </div>
        <p class="p1" v-show="stepIndex === 3">修改成功</p>
        <p class="p2" v-show="stepIndex === 3">当前已验证手机号码：<span>13719394854</span></p>
        <p class="p3" v-show="stepIndex === 3">您可以用该手机号码登录系统</p>
      </div>
      <div class="update_footer">
        <Button @click="updatePhone = false">取消</Button>
        <Button type="primary" @click="gotoNext(1)" v-if="stepIndex === 1">下一步</Button>
        <Button type="primary" @click="gotoNext(2)" v-if="stepIndex === 2">下一步</Button>
        <Button type="primary" @click="updatePhone = false" v-if="stepIndex === 3">确定</Button>
      </div>
    </Modal>
    <!--修改密码-->
    <Modal v-model="updateDraw" width="720" class="updatePwd" footer-hide>
      <div class="update_header">
        <h4>修改密码</h4>
      </div>
      <div class="update_cont">
        <Form ref="formData" :model="formPwd" :rules="ruleValidate" label-position="left">
          <FormItem prop="oldPwd" label="原登录密码">
            <Input v-model="formPwd.oldPwd" type="password" placeholder="请输入原登录密码" class="w330" :maxlength="20"/>
          </FormItem>
          <FormItem prop="pwd1" label="设置新密码">
            <Input v-model="formPwd.pwd1" type="password" placeholder="设置新密码" class="w330" :maxlength="20"/>
          </FormItem>
          <FormItem prop="pwd2" label="确认新密码">
            <Input v-model="formPwd.pwd2" type="password" placeholder="确认新密码" class="w330" :maxlength="20"/>
          </FormItem>
        </Form>
      </div>
      <div class="update_footer">
        <Button @click="updateDraw = false">取消</Button>
        <Button type="primary" @click="confirmNewPwd">确定</Button>
      </div>
    </Modal>
    <!--修改头像-->
    <Modal v-model="uploadImg" width="702" class="uploadImg" footer-hide>
      <div class="uploadImg_header">
        <h4>更换头像</h4>
      </div>
      <div class="uploadImg_cont" v-if="!hasUrl">
        <div class="left">
          <input type="file" id="change" @change="change">
          <div class="left_cont">
            <img src="../../assets/img/icon/img_gray_cloud.png" alt="">
            <p>支持jpg、png格式的图片，大小不超过2M</p>
            <div class="button">
              <img src="../../assets/img/icon/table_ic_gray_upload.png" alt="">
              <span>上传图片</span>
            </div>
          </div>
        </div>
        <div class="right">
          <div class="big">
            <div class="circle"></div>
          </div>
          <div class="middle">
            <div class="circle"></div>
          </div>
          <div class="small">
            <div class="circle"></div>
          </div>
          <p>预览</p>
        </div>
      </div>
      <div class="uploadImg_cont" v-else>
        <div class="left">
          <vue-cropper
            ref="cropper"
            :img="option.img"
            :outputSize="option.size"
            :outputType="option.outputType"
            :info="true"
            :full="option.full"
            :canMove="option.canMove"
            :canMoveBox="option.canMoveBox"
            :original="option.original"
            :autoCrop="option.autoCrop"
            :autoCropWidth="option.autoCropWidth"
            :autoCropHeight="option.autoCropHeight"
            :fixedBox="option.fixedBox"
            @realTime="realTime"
          ></vue-cropper>
        </div>
        <div class="right">
          <div class="big">
            <img class="circle" :src="url"/>
          </div>
          <div class="middle">
            <img class="circle" :src="url"/>
          </div>
          <div class="small">
            <img class="circle" :src="url"/>
          </div>
          <p>预览</p>
        </div>
      </div>
      <div class="uploadImg_footer">
        <Button @click="uploadImg = false">取消</Button>
        <Button type="primary" @click="save()">保存</Button>
      </div>
    </Modal>
  </div>
</template>
<script>
  import NavBar from '_c/navbar'
  import {VueCropper} from 'vue-cropper'
  import {oss} from '@/libs/uploadFile'
  import axios from 'axios'
  import myTask from './myTaskCom'
  import {
    editAvatar,
    getUserInfo,
    passWordRetPwd,
    getUserMenuList,
    queryList,
    getCode,
    codeVerif,
    sendForEditM,
    editMobile,
    signOut
  } from '@/api/api-user'
  import md5 from "js-md5";
  import {JSEncrypt} from "jsencrypt";
  import env from '@/config/env'

  export default {
    components: {
      NavBar,
      VueCropper,
      myTask
    },
    data() {
      // 验证手机号码
      const validatePhone = (rule, value, callback) => {
        if (value === "") {
          callback(new Error("请输入手机号码"));
        } else if (!/^1[34578]\d{9}$/.test(value)) {
          callback(new Error("手机号码不合法"));
        } else {
          callback();
        }
      };
      // 验证验证码
      const validateCode = (rule, value, callback) => {
        let bool = false;
        if(this.stepIndex === 1){
          bool  = this.phoneList.some(item => item === this.setUserInfo.phoneNumber);
        }else if(this.stepIndex === 2){
          bool  = this.phoneList.some(item => item === this.phoneData2.phone);
        }
        if(!bool){
          callback('请先获取验证码');
        }else{
          if (value === "") {
            callback(new Error("请输入验证码"));
          } else if (!/^[0-9]{6}$/.test(value)) {
            callback(new Error("验证码必须为6位数字"));
          } else {
            callback();
          }
        }
      };
      return {
        flag: 1,
        updateDraw: false,
        uploadImg: false,
        updatePhone: false,
        stepIndex: 1,
        hasUrl: false,
        previews: {},
        option: {
          img: '',
          outputSize: 1, //剪切后的图片质量（0.1-1）
          full: true,//输出原图比例截图 props名full
          outputType: 'png',
          canMove: true,
          original: false,
          canMoveBox: true,
          autoCrop: true,
          autoCropWidth: 280,
          autoCropHeight: 280,
          fixedBox: true,
          mode: 'cover',
        },
        phoneData: {
          code: ''
        },
        phoneData2: {
          phone: '',
          code: ''
        },
        rulePhoneData: {
          code: [{validator: validateCode, trigger: "blur"}]
        },
        rulePhoneData2: {
          phone: [{validator: validatePhone, trigger: "change"}],
          code: [{validator: validateCode, trigger: "blur"}]
        },
        // 发送验证码是否显示
        isCodeShow: true,
        // 是否可以发送
        canSendCode: false,
        formPwd: {
          oldPwd: '',
          pwd1: '',
          pwd2: ''
        },
        ruleValidate: {
          oldPwd: [
            {required: true, message: "请输入原登录密码", trigger: "blur"}
          ],
          pwd1: [
            {required: true, message: "请输入新密码", trigger: "blur"}
          ],
          pwd2: [
            {required: true, message: "请确认新密码", trigger: "blur"}
          ],
        },
        headerImage: '',
        picValue: '',
        cropper: '',
        url: '',
        uploadHost: '',
        uploadData: {},
        curCompany: {},
        CONS: "xqc1254548787244",
        place1: {
          ip: '',
          city: '',
          createTime: ''
        },
        place2: {
          ip: '',
          city: '',
          createTime: ''
        },
        codeCount: 60,
        secondStep: false,
        phoneList: []
      }
    },
    created() {
      this.setUserInfo.companys.forEach(item => {
        if (item.compId === this.companyId) {
          this.curCompany = item;
        }
      })
      if(this.$route.params.updateDraw){
        this.updateDraw = true;
      }
      if(this.$route.params.updatePhone){
        this.updatePhone = true;
      }
      this.queryList();

    },
    beforeRouteEnter(to,from,next){
      if(from.name === 'Index'){
        next(vm=>{
          vm.flag = from.query.index ==2 ? 2 : 1
        })
      }else if(!from.name){
        // 刷新页面
        next(vm=>{
          vm.flag = from.query.index ==2 ? 2 : 1
        })
      }else{ 
        next(vm=>{
          vm.flag = from.name === 'BusinessInfoDetail'? 2 : 1
        })

      }
    },
    mounted() {
      
    },
    computed: {
      setUserInfo() {
        return this.$store.getters.userInfo;
      },
      companyId() {
        return this.$store.getters.companyId;
      }
    },
    methods: {
      // 登出
      async logout() {
        let res = await signOut();
        console.log(res);
        if (res.code === 10200) {
          // 清空token
          sessionStorage.clear()
          this.$router.push({name: 'Login'})
        }
      },
      //切换主体
      companyChange(item) {
        console.log(item);
        if (item.compId === this.companyId) {
          this.$Message.error('当前已经是目标主体');
          return;
        }
        this.$router.push({name: 'Index', params: {item}})
      },
      //tab切换
      changeFlag(val) {
        this.flag = val;
      },
      //获取IP
      async queryList() {
        let params = {
          userId: this.setUserInfo.id
        }
        let res = await queryList(params);
        if (res.isSuccess) {
          if (res.data.length === 1) {
            this.place1 = res.data[0];
          } else {
            this.place1 = res.data[0];
            this.place2 = res.data[1];
          }
        }
      },
      // 发送验证码
      async sendCode(val) {
        if (val == 2 && this.phoneData2.phone === this.setUserInfo.phoneNumber) {
          this.$Message.error('新手机号码不能与原号码一致');
          return;
        }
        let data = {
          mobile: val == 1 ? this.setUserInfo.phoneNumber : this.phoneData2.phone,
          channel: "pc",
          appName: '薪结算',
          isNew: val == 1 ? 'old' : 'new'
        };
        let isChecked = true;
        if (val == 2) {
          //验证手机格式正确才能发短信
          // if (this.phoneData2.phone === "") {
          //   this.$Message.error('请输入手机号码');
          //   return;
          // } else if (!/^1[34578]\d{9}$/.test(this.phoneData2.phone)) {
          //   return;
          // }
          this.$refs['phoneData2'].validateField('phone', (bool) => {
            //bool是错误信息 如果有错误信息 就不行
            if (bool) {
              isChecked = false;
            }
          })
        }
        if (!isChecked) {
          return;
        }
        // if(val == 2){
        //   if (this.phoneData2.phone === "") {
        //     this.$Message.error("请输入手机号码");
        //   } else if (!/^1[34578]\d{9}$/.test(this.phoneData2.phone)) {
        //     this.$Message.error("手机号码不合法");
        //   }
        //   return;
        // }

        let res = await sendForEditM(data);
        if (res.code !== 10200) {
          this.$Message.error(res.message);
          return;
        }
        if (val == 1) {
          this.phoneList.push(this.setUserInfo.phoneNumber)
        } else {
          this.phoneList.push(this.phoneData2.phone)
        }
        this.isCodeShow = false;
        this.timer = setInterval(() => {
          this.codeCount--;
          if (this.codeCount <= 0) {
            this.isCodeShow = true;
            this.codeCount = 60;
            clearInterval(this.timer);
          }
        }, 1000);
      },
      gotoNext(val) {
        if (val == 1) {
          let formValidate = this.$refs.phoneData;
          formValidate.validate((valid) => {
            if (valid) {
              this.codeVerif();
            }
          })
        } else {
          let isChecked = true;
          this.$refs['phoneData2'].validateField('phone', (bool) => {
            //bool是错误信息 如果有错误信息 就不行
            if (bool) {
              isChecked = false;
            }
          })
          if (!isChecked) {
            return;
          }
          let formValidate = this.$refs.phoneData2;
          formValidate.validate((valid) => {
            if (valid) {
              this.editMobile();
            }
          })
        }
      },
      // 修改手机号码第一步
      async codeVerif() {
        let data = {
          mobile: this.setUserInfo.phoneNumber,
          code: this.phoneData.code
        }
        let res = await codeVerif(data);
        console.log(res);
        if (res.code === 10200) {
          this.stepIndex++;
          this.isCodeShow = true;
          this.codeCount = 60;
          clearInterval(this.timer);
          this.$refs.phoneData.resetFields()
          this.$refs.phoneData2.resetFields()
        } else {
          this.$Message.error(res.message);
        }
      },
      //修改手机号码第二步
      async editMobile() {
        let data = {
          userId: this.setUserInfo.id,
          mobile: this.setUserInfo.phoneNumber,
          newMobile: this.phoneData2.phone,
          code: this.phoneData2.code
        }
        let res = await editMobile(data);
        console.log(res);
        if (res.code === 10200) {
          this.$Message.success('修改成功,请重新登录');
          this.updatePhone = false;
          this.logout()
        } else {
          this.$Message.error(res.message);
        }
      },
      //实时获取截图
      realTime() {
        this.$refs.cropper.getCropData((data) => {
          this.url = data;
        })
      },
      //上传阿里云
      oss() {
        oss('jietu.jpg').then(res => {
          this.uploadHost = res.host
          this.uploadData = res
          this.uploadFile();
        })
      },
      //打开修改图片的弹窗
      showImg() {
        this.hasUrl = false;
        this.uploadImg = true;
      },
      //修改密码的弹窗
      updatePwd() {
        this.$refs.formData.resetFields();
        this.updateDraw = true;
      },
      encryptedPSW(password) {
        // 新建JSEncrypt对象
        let encryptor = new JSEncrypt();
        // 公钥
        let pubkey = env.pubKey
        // 公钥加密
        encryptor.setPublicKey(pubkey);
        // 加密数据
        return encryptor.encrypt(password);
      },
      //修改手机
      changePhone() {
        clearInterval(this.timer);
        this.isCodeShow = true;
        this.codeCount = 60;
        this.updatePhone = true;
        this.phoneList = [];
        this.stepIndex = 1;
        this.$refs.phoneData.resetFields()
        this.$refs.phoneData2.resetFields()
      },
      //修改密码
      async passWordRetPwd() {
        let password = md5.hex(this.formPwd.oldPwd + this.CONS);
        let newPassword = md5.hex(this.formPwd.pwd1 + this.CONS);
        let data = {
          mobile: this.setUserInfo.phoneNumber,
          password: this.encryptedPSW(password),
          newPassword: this.encryptedPSW(newPassword),
        }
        let res = await passWordRetPwd(data);
        if (res.code === 10200) {
          this.$Message.success('修改成功,请重新登录');
          this.updateDraw = false;
          this.logout()
        } else {
          this.$Message.error(res.message);
        }
      },
      //确认修改
      confirmNewPwd() {
        let formValidate = this.$refs.formData
        formValidate.validate((valid) => {
          if (valid) {
            let bool = this.formPwd.oldPwd.length >= 6 && this.formPwd.pwd1.length >= 6 && this.formPwd.pwd2.length >= 6;
            if (!bool) {
              this.$Message.error('密码必须6-20位组成');
              return;
            }
            if (this.formPwd.pwd1 !== this.formPwd.pwd2) {
              this.$Message.error('两次新密码输入不一致');
              return;
            } else {
              if (this.formPwd.oldPwd === this.formPwd.pwd1) {
                this.$Message.error('新旧密码不能一样');
                return;
              }
            }
            this.passWordRetPwd();
          }
        })
      },
      //保存截图
      save() {
        if (!this.hasUrl) {
          this.$Message.error('请先上传一张图片');
          return;
        }
        this.oss();
      },
      //base64转为二进制
      dataURLtoBlob(dataurl) {
        var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
          bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
        while (n--) {
          u8arr[n] = bstr.charCodeAt(n);
        }
        return new Blob([u8arr], {type: mime});
      },
      //上传截图到阿里云
      async uploadFile() {
        var fileObj = this.url;
        var form = new FormData();
        var url = this.uploadHost;
        fileObj = this.dataURLtoBlob(fileObj);
        for (var key in this.uploadData) {
          form.append(key, this.uploadData[key]);
        }
        form.append("file", fileObj);
        let res = await axios.post(url, form)
        if (res.data.isSuccess) {
          this.editAvatar(res.data.data.url)
        }
      },
      //修改图片
      async editAvatar(avatar) {
        let data = {
          avatar
        }
        let res = await editAvatar(data);
        if (res.isSuccess) {
          this.uploadImg = false;
          let res = await getUserInfo()
          if (res.isSuccess) {
            this.hasUrl = false;
            this.$store.dispatch('setUserInfo', res.data);
          }
        }
      },
      //获取本地图片url
      getObjectURL(file) {
        var url = null;
        if (window.createObjectURL != undefined) { // basic
          url = window.createObjectURL(file);
        } else if (window.URL != undefined) { // mozilla(firefox)
          url = window.URL.createObjectURL(file);
        } else if (window.webkitURL != undefined) { // webkit or chrome
          url = window.webkitURL.createObjectURL(file);
        }
        return url;
      },
      //获取本地图片
      change(e) {
        let files = e.target.files || e.dataTransfer.files;
        if (!files.length) return;
        let size = files[0].size;
        let maxSize = 1024 * 1024 * 2;
        if (size > maxSize) {
          this.$Message.error('上传的图片不能超过2M~');
          return;
        }
        let type = files[0].type;
        if (type !== "image/png" && type !== "image/jpeg") {
          this.$Message.error('只支持上传png和jpg格式的图片');
          return;
        }
        this.picValue = files[0];
        this.option.img = this.getObjectURL(this.picValue);
        this.hasUrl = true;
      },
    }
  }
</script>
<style lang="less">
  .basicInformation {
    height: 100%;
    .container {
      height: 100%;
      .tabs {
        padding: 0 0 0 53px;
        display: flex;
        justify-content: flex-start;
        height: 50px;
        background-color: #fff;
        border: 1px solid rgba(227, 234, 242, 1);
        margin-bottom: 16px;
        h4 {
          margin-right: 75px;
          font-size: 14px;
          color: rgba(67, 69, 77, 1);
          font-weight: 400;
          height: 100%;
          line-height: 50px;
          cursor: pointer;
          &.active {
            color: rgba(91, 112, 171, 1);
            border-bottom: 3px solid rgba(91, 112, 171, 1);
          }
        }

      }
      .com_submenu {
        display: flex;
        justify-content: space-between;
        padding: 30px 0 30px 40px;
        background: rgba(255, 255, 255, 1);
        border: 1px solid rgba(227, 234, 242, 1);
        .left {
          display: flex;
          justify-content: flex-start;
          .ghtx {
            position: relative;
            cursor: pointer;
            img {
              width: 80px;
              height: 80px;
              margin-right: 30px;
              border-radius: 50%;
            }
            p {
              display: none;
              position: absolute;
              top: 0;
              left: 0;
              width: 80px;
              height: 80px;
              border-radius: 50%;
              background: rgba(0, 0, 0, .5);
              text-align: center;
              font-size: 14px;
              span {
                display: block;
                width: 80px;
                height: 40px;
                color: rgba(255, 255, 255, 1);
                &:nth-child(1) {
                  line-height: 70px;
                }
              }
            }
            &:hover {
              p {
                display: block;
              }
            }
          }
          .content {
            padding: 14px 0;
            .cont_top, .cont_bottom {
              display: flex;
              justify-content: flex-start;
              align-items: center;
              font-size: 16px;
              h4 {
                color: rgba(67, 69, 77, 1);
                margin-right: 30px;
              }
              p {
                color: rgba(139, 148, 165, 1);
                margin-right: 10px;
                img {
                  width: 10px;
                  height: 14px;
                  margin-right: 0px;
                }
              }
              .ivu-btn {
                width: 80px;
                height: 26px;
                padding: 0px 8px 5px;
                border: 1px solid rgba(234, 235, 244, 1);
                border-radius: 3px;
                color: rgba(91, 112, 171, 1);
                font-size: 12px;
                img {
                  width: 12px;
                  height: 12px;
                  margin-right: 3px;
                  vertical-align: middle;
                }
                span {
                  display: inline-block;
                  vertical-align: middle;
                }
              }
            }
            .cont_top {
              margin-bottom: 12px;
            }
            .cont_bottom {
              font-size: 12px;
              p {
                color: rgba(67, 69, 77, 1);
                margin-right: 60px;
                &:nth-child(3) {
                  margin-right: 10px;
                }
              }
            }
          }
        }
        .right {
          float: right;
          padding: 14px 0;
          div {
            border-left: 1px solid #E3EAF2;
            padding-left: 20px;
            p {
              margin-bottom: 8px;
            }
          }
        }
      }
      .joined {
        min-height: calc(100% - 232px);
        border: 1px solid rgba(227, 234, 242, 1);
        padding: 18px 0 0px 22px;
        background-color: #fff;
        h4 {
          font-size: 16px;
          font-weight: bold;
          color: rgba(51, 51, 51, 1);
          margin-bottom: 34px;
        }
        ul {
          display: flex;
          flex-wrap: wrap;
          li {
            display: flex;
            justify-content: space-between;
            align-items: center;
            width: 400px;
            padding: 15px;
            margin-right: 22px;
            margin-bottom: 20px;
            border: 1px solid rgba(227, 230, 237, 1);
            border-radius: 4px;
            cursor: pointer;
            &:hover {
              box-shadow: 0px 2px 8px rgba(0, 0, 0, 0.06);
            }
            div {
              display: flex;
              justify-content: flex-start;
              align-items: center;
              img {
                width: 16px;
                height: 16px;
                margin-right: 12px;
              }
              p {
                display: flex;
                align-items: center;
                font-size: 14px;
                .main_com {
                  display: inline-block;
                  width: 185px;
                  overflow: hidden; //超出的文本隐藏
                  text-overflow: ellipsis; //溢出用省略号显示
                  white-space: nowrap; //溢出不换行
                  font-size: 14px;
                  color: rgba(67, 69, 77, 1);
                }
                .cur {
                  display: inline-block;
                  line-height: 20px;
                  color: rgba(255, 255, 255, 1);
                  background: rgba(170, 174, 223, 1);
                  border-radius: 4px;
                  padding: 0 2px;
                  margin-left: 8px;
                }
              }
            }
            span {
              font-size: 12px;
              color: rgba(139, 148, 165, 1);
            }
          }
        }
      }
    }
  }

  .updatePhone {
    .ivu-modal-content {
      border-radius: 0;
      .ivu-modal-body {
        padding: 1px;
        .update_header {
          background: rgba(242, 243, 247, 1);
          h4 {
            padding-left: 20px;
            line-height: 56px;
            font-size: 14px;
            color: rgba(67, 69, 77, 1);
            font-weight: bold;
          }
        }
        .update_cont {
          padding: 81px 186px 110px;
          .step {
            display: flex;
            justify-content: space-between;
            margin-bottom: 40px;
            div {
              display: flex;
              flex-direction: column;
              justify-content: space-between;
              align-items: center;
              span {
                &.circle {
                  width: 22px;
                  height: 22px;
                  border-radius: 50%;
                  background: rgba(212, 215, 233, 1);
                  font-size: 14px;
                  color: rgba(255, 255, 255, 1);
                  /*font-weight: bold;*/
                  text-align: center;
                  line-height: 22px;
                  margin-bottom: 11px;
                  &.active {
                    background: rgba(103, 110, 187, 1);
                  }
                }
              }
            }
            .line {
              width: 83px;
              height: 2px;
              background: rgba(212, 215, 233, 1);
              border-radius: 2px;
              margin-top: 10px;
              &.active {
                background: rgba(103, 110, 187, 1);
              }
            }
          }
          .p0 {
            text-align: center;
            font-size: 14px;
            color: rgba(67, 69, 77, 1);
            margin-bottom: 14px;
          }
          .w204 {
            width: 204px;
            height: 40px;
            margin-right: 16px;
          }
          .ivu-btn {
            width: 110px;
            height: 40px;
          }
          .content {
            text-align: center;
            img {
              width: 56px;
              height: 56px;
              margin-bottom: 12px;
            }
          }
          .p1 {
            text-align: center;
            font-size: 16px;
            color: rgba(75, 176, 141, 1);
            margin-bottom: 15px;
          }
          .p2 {
            text-align: center;
            color: rgba(67, 69, 77, 1);
            font-size: 14px;
            margin-bottom: 10px;
            span {
              color: rgba(82, 118, 222, 1);
            }
          }
          .p3 {
            text-align: center;
            font-size: 12px;
            color: rgba(139, 148, 165, 1);
          }
        }
        .update_footer {
          display: flex;
          justify-content: flex-end;
          padding: 12px 20px 12px 0;
          border-top: 1px solid #E3E6ED;
          .ivu-btn {
            margin-left: 10px;
          }
        }
      }
    }
  }

  .updatePwd {
    .ivu-modal-content {
      border-radius: 0;
      .ivu-modal-body {
        padding: 1px;
        .update_header {
          background: rgba(242, 243, 247, 1);
          h4 {
            padding-left: 20px;
            line-height: 56px;
            font-size: 14px;
            color: rgba(67, 69, 77, 1);
            font-weight: bold;
          }
        }
        .update_cont {
          padding: 86px 143px 68px;
          border-bottom: 1px solid #e8eaec;
          .ivu-form-item-label {
            font-size: 14px;
            height: 40px;
            line-height: 22px;
          }
          .w330 {
            width: 330px;
            height: 40px;
          }
          .ivu-form-item-error-tip {
            left: 100px;
          }
        }
        .update_footer {
          display: flex;
          justify-content: flex-end;
          padding: 12px 20px 12px 0;
          .ivu-btn {
            margin-left: 10px;
          }
        }
      }
      .ivu-modal-footer {
        border-top: 0;
        .ivu-btn-text {
          background-color: #fff;
          border: 1px solid rgba(207, 209, 224, 1);
        }
      }
      .ivu-icon-ios-close {
        color: #000;
      }
    }
  }

  .uploadImg {
    .ivu-modal-content {
      border-radius: 0;
      .ivu-modal-body {
        padding: 1px;
        .uploadImg_header {
          background: rgba(242, 243, 247, 1);
          h4 {
            padding-left: 20px;
            line-height: 56px;
            font-size: 14px;
            color: rgba(67, 69, 77, 1);
            font-weight: bold;
          }
        }
        .uploadImg_cont {
          padding: 20px 110px;
          border-bottom: 1px solid #ccc;
          display: flex;
          justify-content: space-between;
          .left {
            position: relative;
            width: 350px;
            height: 350px;
            background-color: rgba(242, 243, 247, 1);
            #change {
              position: absolute;
              left: 0;
              top: 0;
              width: 100%;
              height: 100%;
              opacity: 0;
            }
            .left_cont {
              display: flex;
              flex-direction: column;
              justify-content: space-between;
              align-items: center;
              padding-top: 52px;
              img {
                width: 120px;
                height: 120px;
              }
              p {
                margin-bottom: 34px;
                color: rgba(139, 148, 165, 1);
              }
              .button {
                display: flex;
                align-items: center;
                padding: 5px;
                background-color: #fff;
                img {
                  width: 16px;
                  height: 16px;
                }
              }
            }
          }
          .right {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            width: 128px;
            height: 350px;
            padding: 24px 0;
            background: rgba(242, 243, 247, 1);
            .big {
              .circle {
                width: 80px;
                height: 80px;
                background: rgba(221, 223, 232, 1);
                border: 2px solid rgba(255, 255, 255, 1);
                border-radius: 50%;
              }
            }
            .middle {
              .circle {
                width: 60px;
                height: 60px;
                background: rgba(221, 223, 232, 1);
                border: 2px solid rgba(255, 255, 255, 1);
                border-radius: 50%;
              }
            }
            .small {
              .circle {
                width: 40px;
                height: 40px;
                background: rgba(221, 223, 232, 1);
                border: 2px solid rgba(255, 255, 255, 1);
                border-radius: 50%;
              }
            }
            p {
              color: rgba(139, 148, 165, 1)
            }
          }
        }
        .uploadImg_footer {
          display: flex;
          justify-content: flex-end;
          padding: 12px 20px 12px 0;
          .ivu-btn {
            margin-left: 10px;
          }
        }
      }
    }
    .ivu-icon-ios-close {
      color: #000;
    }
  }
</style>