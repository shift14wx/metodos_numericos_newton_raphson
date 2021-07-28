<template>

    <v-simple-table>
      <template v-slot:default>
        <thead>
        <tr>
          <th class="text-left">
           i
          </th>
          <th class="text-left">
           Xn-1
          </th>
          <th class="text-left">
            y = f(Xn-1)
          </th>
          <th class="text-left">
            dy/dx
          </th>
          <th class="text-left">
            Xn
          </th>
          <th class="text-left">
            Error
          </th>
        </tr>
        </thead>
        <tbody>
        <tr
            v-for="(item, i) in value.table_data"
            :key="i+'a'"
            :class="{'bg-black':(i==valores.table_data.length)}"
        >
          <td>{{ item[0] }}</td>
          <td>{{ item[1] }}</td>
          <td>{{ item[2] }}</td>
          <td>{{ item[3] }}</td>
          <td>{{ item[4] }}</td>
          <td>{{ item[5] }}</td>
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
  }
}

</script>

<style scoped>

</style>