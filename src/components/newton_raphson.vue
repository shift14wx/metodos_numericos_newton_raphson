<template>
  <v-container>
    <v-row>
      <v-col>
        <v-form v-model="form" ref="form">
          <v-card>
            <v-card-title>Calcular</v-card-title>
            <v-card-text>
              <v-container>
                <v-text-field
                    v-model="expresion"
                    outlined
                    placeholder="Ingrese la función"
                    label="Función"
                    :prepend-inner-icon="'mdi-function'"
                    :rules="rules"
                />
                <v-row>
                  <v-col cols="6">
                    <v-text-field
                        v-model="variable"
                        outlined
                        placeholder="Ingrese la variable a sustituir en función"
                        label="Variable en función"
                        type="text"
                        :prepend-inner-icon="'mdi-circle-medium'"
                        :rules="rules"
                    />
                  </v-col>
                  <v-col cols="6">
                    <v-text-field
                        v-model="tolerance"
                        outlined
                        placeholder="Ingrese la tolerancia"
                        label="Tolerancia"
                        type="number"
                        :prepend-inner-icon="'mdi-circle-medium'"
                        :rules="[
                        v => !!v || 'Este campo es necesario',
                        v => parseFloat(v) > 0.0 || 'Esta campo no puede ser negativo'
                        ]"
                    />
                  </v-col>
                </v-row>

                <v-row>
                  <v-col cols="4">
                    <v-text-field
                        v-model="limiteInferior"
                        outlined
                        placeholder="Ingrese el limite inferior (X inicial para calcular la raiz)"
                        label="Limite Inferior (X inicial para calcular la raiz)"
                        type="number"
                        :prepend-inner-icon="'mdi-arrow-down-bold'"
                        :rules="rules"
                    />
                  </v-col>

                  <v-col cols="4">
                    <v-text-field
                        v-model="limiteSuperior"
                        outlined
                        placeholder="Ingrese el limite Superior"
                        label="Limite Superior"
                        type="number"
                        :prepend-inner-icon="'mdi-arrow-up-bold'"
                        :rules="rules"
                    />
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                        v-model="incremento"
                        outlined
                        placeholder="Ingrese el incremento"
                        label="Incremento"
                        type="number"
                        :prepend-inner-icon="'mdi-plus'"
                        :rules="[
                        v => !!v || 'Este campo es necesario',
                        v => parseFloat(v) > 0.0 || 'Esta campo no puede ser negativo'
                        ]"
                    />
                  </v-col>
                </v-row>
                <v-btn @click="evaluar" color="success">Calcular</v-btn>

              </v-container>
            </v-card-text>
          </v-card>
        </v-form>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-tabs
            v-model="tab"
            background-color="deep-purple accent-4"
            centered
            dark
            icons-and-text
        >
          <v-tabs-slider></v-tabs-slider>

          <v-tab href="#tab-0">
            Grafico
            <v-icon>mdi-graphql</v-icon>
          </v-tab>

          <v-tab href="#tab-1">
            Calcular Raiz
            <v-icon>mdi-table</v-icon>
          </v-tab>

          <v-tab href="#tab-2">
            Tabla de valores
            <v-icon>mdi-table</v-icon>
          </v-tab>
        </v-tabs>

        <v-tabs-items v-model="tab">
          <v-tab-item
              v-for="(item,i) in items"
              :key="item.titulo"
              :value="'tab-' + i"
          >
            <v-card flat>
              <v-card-title>{{ item.titulo }}</v-card-title>
              <v-card-text>
                <div id="plot" v-if="i===0"></div>

                <tabla v-if="i===1 && valores" v-model="valores"/>
                <tabla2 v-if="i===2 && valoresMedios" v-model="valoresMedios"/>

              </v-card-text>
            </v-card>
          </v-tab-item>
        </v-tabs-items>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import {compile, range, derivative, abs, evaluate} from 'mathjs';
import {newPlot} from 'plotly.js';
import Tabla from "./tabla";
import Tabla2 from "./tabla2";

export default {
  components: {Tabla2, Tabla},
  data() {
    return {
      expresion: "x^3-2x-5",
      tab: null,
      tolerance:null,
      limiteInferior: null,
      limiteSuperior: null,
      incremento: null,
      variable:"x",
      valores:null,
      valoresMedios:null,
      form: false,
      rules: [
        v => !!v || 'Este campo es necesario',
      ],
      items: [
        {
          titulo: "Grafico",
          icono: ""
        },
        {
          titulo: "Calcular raiz",
          icono: ""
        },
        {
          titulo: "Tabla de valores",
          icono: ""
        }
      ],
    }
  },
  methods: {
    calcular_entre_2_cifras(){
      let datos = [];
      let valor=0.0;
      let i = parseFloat(this.limiteInferior);

      while (parseFloat(i)<= parseFloat(this.limiteSuperior)){
        console.log(i);
        valor = this.evaluate_expression(this.expresion,i);
        i+=parseFloat(this.incremento);
        datos.push([i,valor]);
      }


      this.valoresMedios = datos;
      console.log(this.valoresMedios);
    },
    draw() {
      const expr = compile(this.expresion);
      const xValues = range(-5, 5, 0.1).toArray()
      const yValues = xValues.map(function (x) {
        return expr.evaluate({x: x})
      })
      // render the plot using plotly
      const trace1 = {
        x: xValues,
        y: yValues,
        type: 'scatter'
      }

      const data = [trace1]
      newPlot('plot', data)

    },
    evaluar() {
      if (this.$refs.form.validate()){
        this.draw();
        this.valores = this.newton_raphson();
        this.calcular_entre_2_cifras();
      }
    },
    calculate_derivative() {
      try {
        return derivative(this.expresion, this.variable);
      } catch (e) {
        console.log("calculate_derivative");
        console.log(e);
      }
    },
    newton_raphson() {
      this.valores=null;
      let result_function = 0;
      let result_derivative = 0;
      let previous_x = 0;
      let last_root = 0;
      let root = this.limiteInferior;
      let i = 0;
      let error = 100;
      let data = [];
      let headers = ["i","Xn-1","y = f(Xn-1)","dy/dx","Xn","Error"];

      while (error > this.tolerance) {
        i++;

        previous_x = root;

        result_function = this.evaluate_expression(this.expresion,previous_x);
        result_derivative = this.evaluate_expression(this.calculate_derivative().toString(),previous_x);

        root = previous_x - (result_function/result_derivative);
        last_root = previous_x;

        error = abs((root - last_root)/root * 100);

        if (i == 1) {
          data.push([i, previous_x,result_function,result_derivative, root, "--"]);
        } else {
          data.push([i, previous_x,result_function,result_derivative, root, error]);
        }
      }

      let result = {
        res_root: root,
        res_iterations: i,
        res_error: error,
        table_headers: headers,
        table_data: data
      }

      console.log(result);
      return result;
    },
    evaluate_expression(expression, value) {
      try {
        let re = new RegExp(this.variable, "g");
        let new_expression = expression.replace(re, `(${value})`);
        return evaluate(new_expression);
      } catch (e) {
        console.log("evaluate_expression");
        console.log(expression);
        console.log(e);
      }
    }
  },
  computed: {

  },
  mounted() {

  }
}
</script>
