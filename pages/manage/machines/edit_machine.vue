
<template>
    <v-card>
      <v-alert
        v-model="danger"
        dismissible
        :type=type_api
      >
        {{alt_txt}}
      </v-alert>
        <v-card-title
          class="grey lighten-4 py-4 title"
        >
        <v-flex xs7 >
          แก้ไขข้อมูลพาหนะ
        </v-flex>
        <v-flex xs2 >
          <v-btn
            color="green lighten-2"
            flat
            @click="isEditing = !isEditing"
          >
            <i v-if="isEditing" class="fas fa-times fa-2x"></i>
            <i v-else class="fas fa-edit fa-2x "></i>
          </v-btn>
        </v-flex>
        <v-flex xs2 >
          <v-dialog v-model="conf_del" persistent max-width="290">
            <v-btn slot="activator" flat color="red lighten-2"><i class="fas fa-trash-alt fa-2x"></i></v-btn>
            <v-card>
              <v-card-title class="headline">ยืนยันการลบข้อมูล</v-card-title>
              <v-card-text>ต้องการลบข้อมูล {{mc_code}}<br> ใช่หรือไม่?</v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red lighten-2" flat @click.native="conf_del = false">ไม่ใช่</v-btn>
                <v-btn color="primary" flat @click="machines_del()">ใช่</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-flex>
        </v-card-title>
        <v-container grid-list-sm class="pa-4">
          <v-layout row wrap>
             <v-flex xs12 >
               <v-layout>
                 <v-flex
                  xs4
                ></v-flex>
                <v-flex
                  xs4
                  d-flex
                >
                  <v-card flat tile class="d-flex">
                    <v-img
                      :src="img_img"
                      :lazy-src="img_img"
                      aspect-ratio="1"
                      class="grey lighten-2"
                    >
                      <v-layout
                        slot="placeholder"
                        fill-height
                        align-center
                        justify-center
                        ma-0
                      >
                        <v-progress-circular indeterminate color="grey lighten-5"></v-progress-circular>
                      </v-layout>
                    </v-img>
                  </v-card>
                </v-flex>
              </v-layout>
            </v-flex>
            <v-flex xs12 >
              <v-layout align-center>
                <v-text-field
                  :disabled="true"
                  maxlength="50"
                  counter
                  prepend-icon="fas fa-user"
                  label="ชื่อเจ้าของพาหนะ"
                  v-model="std_name"
                ></v-text-field>
              </v-layout>
            </v-flex>
            <v-flex xs12 >
              <v-layout align-center>
                <v-text-field
                  :disabled="!isEditing"
                  :rules="[rules.required]"
                  maxlength="50"
                  counter
                  prepend-icon="fas fa-id-card-alt fa-2x"
                  placeholder="ทะเบียนรถ"
                  name="mc_code"
                  v-model="mc_code"
                ></v-text-field>
              </v-layout>
            </v-flex>
            <v-flex xs12 >
              <v-layout align-center>
                <v-text-field
                  :disabled="!isEditing"
                  :rules="[rules.required]"
                  maxlength="100"
                  counter
                  placeholder="แบรนด์รถ"
                  prepend-icon="fas fa-car"
                  name="mc_brand"
                  v-model="mc_brand"
                ></v-text-field>
              </v-layout>
            </v-flex>
            <v-flex xs12 >
              <v-layout align-center>
                <v-text-field
                  :disabled="!isEditing"
                  maxlength="100"
                  counter
                  prepend-icon="fas fa-tachometer-alt"
                  placeholder="รุ่นรถ"
                  name="mc_series"
                  v-model="mc_series"
                ></v-text-field>
              </v-layout>
            </v-flex>
           
          </v-layout>
        </v-container>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn flat color="red lighten-2" @click="machine()">ย้อนกลับ</v-btn>
          <v-btn flat color="primary" :disabled="!isEditing" @click="machine_update(mc_id)">บันทึก</v-btn>
        </v-card-actions>
    </v-card>
</template>

<script>
    export default {
        layout: 'manage',

        data () {
            return {
              mc_id:"",
              mc_code:"",
              mc_brand:"",
              mc_series: '',
              std_id: '',
              img_img:"",

              type_api:"",
              danger:false,
              conf_del:false,
              isEditing:null,
              rules: {
                    required: value => !!value || 'ห้ามว่าง.',
                    // counter: value => value.length <= 10 || 'เต็ม 10 ตัวอักษร',
              },
            }
        },
        async created(){
          this.sh_machine()
        },
        methods:{
          conf_del(){this.conf_del=true},
          async machines_del(){
            let res=await this.$http.get('/machine/machine_del/'+this.$route.query.mc_id)
            if(res.data.ok==true){this.$router.push({name:"manage-machines"})}
            else{this.danger=true,this.alt_txt=res.data.txt,this.type_api=res.data.alt}
          },
          async sh_machine(){
            let res=await this.$http.get('/machine/sh_machine/'+this.$route.query.mc_id)
            // console.log(res.data.datas)
            this.mc_id=this.$route.query.mc_id
            this.mc_code=res.data.datas[0].mc_code
            this.mc_brand=res.data.datas[0].mc_brand
            this.mc_series=res.data.datas[0].mc_series
            this.std_prename=res.data.datas[0].std_prename
            this.std_name=res.data.datas[0].std_name
            this.std_lastname=res.data.datas[0].std_lastname
            this.img_img=res.data.datas[0].img_img
          },
          async machine_update(mc_id){
            //console.log("mc_id"+mc_id)
            let res=await this.$http.post("/machine/machine_update",{
              mc_code:this.mc_code,
              mc_brand:this.mc_brand,
              mc_series:this.mc_series,
              std_id:this.std_id,
              mc_id:mc_id,
            })
            console.log(res.data)
              if(res.data.ok==true){this.danger=true,this.alt_txt=res.data.txt,this.type_api=res.data.alt,this.sh_machine()}
             else{this.danger=true,this.alt_txt=res.data.txt,this.type_api=res.data.alt}
          },
          machine(){
            this.$router.push({name:"manage-machines"})
          },

        }
    }
</script>