<template>
<v-app class="main_margin">
    <v-dialog
      v-model="dialog"
      persistent
      max-width="600px"
    >
      <template v-slot:activator="{ on, attrs }">
        <v-btn
          style="width:50px"
          color="primary"
          v-bind="attrs"
          v-on="on"
        >
          新增
        </v-btn>
      </template>
      <v-card>
        <v-card-title>
          <span class="headline">新增员工</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-form
              ref="form"
              v-model="valid"
              lazy-validation
            >
              <v-text-field
                v-model="newData.name"
                :counter="10"
                :rules="nameRules"
                label="姓名"
                required
              ></v-text-field>

              <v-text-field
                v-model="newData.age"
                :rules="[v => !!v ]"
                label="年龄"
                required
              ></v-text-field>

              <v-text-field
                v-model="newData.company"
                :rules="[v => !!v ]"
                label="所属企业"
                required
              ></v-text-field>
              <v-text-field
                v-model="newData.id"
                :rules="[v => !!v ]"
                label="工号"
                required
              ></v-text-field>
              <v-text-field
                v-model="newData.address"
                :rules="[v => !!v ]"
                label="家庭住址"
                required
              ></v-text-field>

              

              <v-btn
                :disabled="!valid"
                color="success"
                class="mr-4"
                @click="save"
              >
                保存
              </v-btn>

              <v-btn
                color="error"
                class="mr-4"
                @click="reset"
              >
                重置
              </v-btn>

              <v-btn
                color="warning"
                @click="cancel"
              >
               取消
              </v-btn>
            </v-form>
          </v-container>
        </v-card-text>

      </v-card>
    </v-dialog>


    <v-data-table
    :headers="headers"
    :items="desserts"
     hide-default-footer
    class="elevation-1"
  ></v-data-table>
</v-app>
</template>
<script>
export default {
 
    data () {
      return {
         valid: true,
         dialog: false,
         nameRules: [
        v => !!v || 'Name is required',
        v => (v && v.length <= 10) || 'Name must be less than 10 characters',
      ],
      newData:{
        name: '',
        age:'',
        company:'',
        id:'',
        address:'',
      },

        headers: [
          {
            text: '姓名',
            align: 'start',
            sortable: false,
            value: 'name',
          },
          { text: '年龄', value: 'age' },
          { text: '所属企业', value: 'company' },
          { text: '工号', value: 'id' },
          { text: '家庭住址', value: 'address' },
        ],
        desserts: [
          {
            name: 'Frozen Yogurt',
            age: 15,
            company: '无锡办公室',
            id: 100024,
            address: '江苏无锡',

          },
          {
            name: 'Ice cream sandwich',
            age: 23,
            company: '无锡办公室',
            id: 100037,
            address: '江苏无锡',

          },
          {
            name: 'Eclair',
            age: 22,
            company: '无锡办公室',
            id: 100023,
            address: '江苏无锡',

          },
          {
            name: 'Cupcake',
            age: 30,
            company: '无锡办公室',
            id: 100067,
            address: '江苏无锡',
          },
          
        ],
      }
    },
    methods:{
      save () {
        this.$refs.form.validate();
        this.desserts.push({...this.newData});
        this.dialog=false
        this.$refs.form.reset()
      },
      reset () {
        this.$refs.form.reset()
      },
      cancel () {
        this.dialog=false
        this.$refs.form.reset()
      },
      

    }
}
</script>
<style scoped>
.main_margin{
  margin: 0 50px;
}
.elevation-1{
  margin-top: 10px;
}
</style>