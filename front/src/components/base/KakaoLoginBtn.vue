<template>
  <button @click="kakaoLogin" style="coursor: pointer">
    <!-- 원본 이미지 -->
    <!-- <svg
    xmlns="http://www.w3.org/2000/svg"
    width="55"
    height="55"
    viewBox="0 0 55 55">
    <g id="그룹_247" data-name="그룹 247" transform="translate(-237 -406)">
        <g id="구성_요소_2" data-name="구성 요소 2" transform="translate(237 406)">
        <g id="타원_6" data-name="타원 6" fill="#ffd800" stroke="#eee" stroke-width="1">
            <circle cx="27.5" cy="27.5" r="27.5" stroke="none"/>
            <circle cx="27.5" cy="27.5" r="27" fill="none"/>
        </g>
        </g>
        <g id="kakaotalk_1_" data-name="kakaotalk (1)" transform="translate(254 424)">
            <path
                id="패스_288"
                data-name="패스 288"
                d="M76.01,74.216h1.229l-.615-1.746Z"
                transform="translate(-68.211 -65.034)"/>
            <path
                id="패스_289"
                data-name="패스 289"
                d="M10.671,0C4.777,0,0,3.767,0,8.414c0,3,2,5.643,5,7.129-.163.563-1.051,3.626-1.086,3.867,0,0-.022.181.1.249a.322.322,0,0,0,.254.015c.337-.047,3.9-2.546,4.515-2.976a13.5,13.5,0,0,0,1.89.128c5.894,0,10.671-3.767,10.671-8.414S16.565,0,10.671,0ZM5.39,7.108c-.013,1.19.01,2.442-.009,3.614-.006.375-.222.486-.513.593a.193.193,0,0,1-.1.007c-.333-.066-.6-.185-.607-.6-.024-1.171.007-2.425-.009-3.615-.282-.011-.684.011-.946,0a.577.577,0,0,1-.6-.6.565.565,0,0,1,.606-.6c.962-.014,2.155-.014,3.117,0a.561.561,0,0,1,.605.6.578.578,0,0,1-.6.6C6.074,7.119,5.673,7.1,5.39,7.108Zm5.171,4.15a.948.948,0,0,1-.392.085.505.505,0,0,1-.513-.272l-.308-.8H7.475l-.308.8a.506.506,0,0,1-.513.272.94.94,0,0,1-.391-.085c-.17-.078-.333-.293-.147-.874L7.593,6.507A.923.923,0,0,1,8.414,5.9a.931.931,0,0,1,.821.608l1.471,3.874C10.894,10.965,10.731,11.184,10.561,11.258Zm3.111,0H11.7a.579.579,0,0,1-.59-.564V6.516a.629.629,0,0,1,1.257,0v3.617h1.308a.565.565,0,1,1,0,1.129Zm4.823-.464a.616.616,0,0,1-1.1.287L15.947,9.17l-.213.213v1.343a.616.616,0,1,1-1.231,0V6.52a.616.616,0,0,1,1.231,0v1.32l1.718-1.718a.476.476,0,0,1,.342-.137.624.624,0,0,1,.605.573.482.482,0,0,1-.134.375l-1.406,1.4,1.516,2.008a.616.616,0,0,1,.118.456Z"/>
        </g>
    </g>
    </svg> -->
    <v-img
        :src="require('@/assets/kakao_login_medium_wide.png')"
    ></v-img>
</button>
</template>

<script>
import axios from 'axios'
import constants from '@/api/constants'
import {mapMutations, mapState} from 'vuex'

export default {
    name: "KakaoLoginBtn",
    data(){
        return {
            data : '',
            constants,
            loginData : {
                username : '',
                password : '',
            },
            submitData : {
                username : '',
                password1 : '',
                password2 : '',
            }
        }
    },
    methods: {
        kakaoLogin() {
            window.Kakao.Auth.login({
                scope : 'profile, account_email',
                success: this.GetMe,
            });
        },
        async GetMe(authObj){
            this.data = window.Kakao.API
            window.Kakao.API.request({
                url:'/v2/user/me',
                success : res => {
                    const kakao_account = res.kakao_account;
                    kakao_account
                    // const userInfo = {
                    //     nickname : kakao_account.profile.nickname,
                    //     email : kakao_account.email,
                    //     password : '',
                    //     account_type : 2,
                    // }
                    // console.log(userInfo)
                    let accessToken = authObj.access_token
                    let form = new FormData()
                    form.append('access_token', accessToken)
                    axios.post(`${this.constants.API_URL}rest-auth/kakao/login/`,form)
                    .then((res)=>{
                      this.SET_TOKEN(res.data.access_token)
                      this.SET_USERNAME(res.data.user.username);
                      this.isFirstLogin()
                    })
                    .catch((err)=>{
                    //   console.log(err)
                    })

                },
                fail : error => {
                    // console.log(error);
                }
            })
        },
        async isFirstLogin(){
            const Token = 'Bearer '+this.userToken
            axios.get(`${this.constants.API_URL}rest-auth/user/`,{
                headers: {
                    Authorization: Token,
                },
                })
                .then((res)=>{
                    if (res.data.is_first === 0 ){
                    // this.$router.push({name:'SubmitProfile'})
                    // console.log('여긴 kakaoBtn', this.$store.state.kakaoCheck)
                    } else{
                        if (res.data.user.is_first == 1 ){
                            this.SET_TYPETOKEN('trainer')
                        } else{
                            this.SET_TYPETOKEN('client')
                        }
                        this.$router.push({ name: 'Home' })
                    }
                })

        },
        login(){
            axios.post(`${this.constants.API_URL}rest-auth/login/`,this.loginData)
            .then((res)=>{
                this.SET_TOKEN(res.data.access_token)
                      this.isFirstLogin()
            })
        },
        submit(){
            axios.post(`${this.constants.API_URL}rest-auth/signup/`,this.submitData)
            .then((res)=>{
                this.SET_TOKEN(res.data.access_token)
                this.isFirstLogin()
            })
        },
        ...mapMutations(['SET_TOKEN','SET_TYPETOKEN',"SET_USERNAME"])
    },
    computed : {
        ...mapState(['userToken'])
    }
}
</script>

<style>

</style>