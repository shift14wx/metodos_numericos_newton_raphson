<template>
  <v-simple-table>
    <template v-slot:default>
      <thead>
      <tr>
        <th class="text-left">
         X
        </th>
        <th class="text-left">
          f(x)
        </th>
      </tr>
      </thead>
      <tbody>
      <tr
          v-for="(item,i) in valores"
          :key="i+'a'"
      >
        <td>{{ item[0] }}</td>
        <td>{{ item[1] }}</td>
      </tr>
      </tbody>
    </template>
  </v-simple-table>
</template>
<script>
export default {
  props:{
    value:{
      required:true,
      default:[]
    }
  },
  data(){
    return{
      innervalores:this.value
    }
  },
  computed:{
    valores:{
      get:function (){
        return this.innervalores;
      },
      set: function (newValue){
        this.innervalores = newValue;
      }
    }
  },

}
</script>

<style scoped>

</style>