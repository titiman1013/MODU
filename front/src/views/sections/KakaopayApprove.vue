<template>
    <div>
      <br><br><br><br><br><br><br><br><br><br>
      <base-section-heading title="카카오 페이 결제가 완료되었습니다." />
          <!-- <div>
            <div><div>주문번호</div><div>{{ res.partner_order_id }}</div></div>
            <div><div>상품명</div><div>{{ res.item_name }}</div></div>
            <div><div>결제금액</div><div>{{ amount }}원</div></div>
            <div><div>결제승인시각</div><div>{{ res.approved_at }}</div></div>
          </div> -->
        <div class="d-flex justify-center">
          <Btn @click="goHome">돌아가기</Btn>
        </div>
  </div>
</template>

<script>
import axios from 'axios'
import constants from '@/api/constants'
import {mapMutations, mapState} from 'vuex'

export default {
    name : 'KakaopayApprove',
    components: {
      Btn: () => import('@/components/base/Btn'),
    },
    data(){
      return {
        constants,
      }
    },
    mounted(){
    },
    methods:{
      goHome() {
        this.$router.push({ name: 'Home' })
      },
      approve(){
          let Token = "Bearer " + this.userToken;
          let form = new FormData()
          // let pg_token = this.$route.query.pg_token
          form.append('tid', this.tid)
          form.append('pg_token', this.$route.query.pg_token)
          form.append('program_id', this.$route.query.program)
          form.append('price_id', this.$route.query.price)
          axios.post(`${this.constants.API_URL}kakaopay/approve/`, form,{
            headers: {
              Authorization: Token,
            },
          })
          .then(()=>{
            this.REMOVE_TID
          })
          .catch(()=>{
            // console.log(err)
          })
      },
      
      ...mapMutations(['REMOVE_TID'])
    },
    created() {
      this.approve()
    },
    computed : {
        ...mapState(['tid', 'userToken'])
    }
}
</script>