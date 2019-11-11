<template>
  <div class="home">
    <br />
    <br />
    <h1>Cotizador de Productos</h1>
    <v-container grid-list-xl fluid>
      <v-layout justify-center>
        <v-flex lg12>
          <v-card>
            <v-container>
              <v-layout>
                <v-flex xs12 md4>
                  <v-select
                    v-model="productoValue"
                    :items="productoSelect"
                    label="Producto"
                    item-text="nombre"
                    item-value="code"
                  ></v-select>
                </v-flex>
                <v-flex xs12 md4>
                  <v-select
                    v-model="paisValue"
                    :items="paisSelect"
                    label="País"
                    item-text="nombre"
                    item-value="code"
                  ></v-select>
                </v-flex>
              </v-layout>
              <v-flex xs12 md4>
                <v-btn dark title="Filtrar contratos" @click="loadRegistro()">Agregar</v-btn>&nbsp;&nbsp;&nbsp;
                <v-btn dark title="Filtrar contratos" @click="borrarCotizacion()">Borrar Cotización</v-btn>
              </v-flex>
              <v-flex xs12 md4 v-if="visible">
                <h1>No existe el producto, no se agregó a la cotización.</h1>
              </v-flex>
            </v-container>
          </v-card>
        </v-flex>
      </v-layout>
      <v-layout justify-center v-if="columnasCotizacion.length > 0">
        <v-flex lg12>
          <v-simple-table>
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">Producto</th>
                  <th class="text-left">País de Compra</th>
                  <th class="text-left">Precio</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="producto in columnasCotizacion" :key="producto.name">
                  <td class="text-left">{{ producto.producto }}</td>
                  <td class="text-left">{{ producto.pais }}</td>
                  <td class="text-left">{{ producto.precio }}</td>
                </tr>
              </tbody>
              <tfoot>
                <tr>
                  <td class="text-left">Valor Total:</td>
                  <td class="text-left">{{valorTotal}}</td>
                </tr>
              </tfoot>
            </template>
          </v-simple-table>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "home",
  components: {},
  data: () => ({
    visible: false,
    info: "",
    posts: [],
    errors: [],
    paisValue: "",
    paisSelect: [],
    productoValue: "",
    productoSelect: [],
    itemsProductos: [],
    columnasCotizacion: [],
    valorDolar: [],
    valorEuro: [],
    valorTotal: 0,
    dolar: 0,
    euro: 0
  }),

  mounted() {
    this.loadData();
    this.loadApiDolar("euro");
    this.loadApiDolar("dolar");
  },
  methods: {
    async loadProductoFilter(nombre, pais) {
      return await axios
        .get("http://localhost:9090/getProduct/" + nombre + "/" + pais)
        .then(function(response) {
          return response.data;
        })
        .catch(function(error) {
          console.log(error);
          return [];
        });
    },
    async loadRegistro() {
      let nombre = "";
      let pais = "";
      if (this.productoValue != "" && this.productoValue != null) {
        nombre = this.productoValue;
      }
      if (this.paisValue != "" && this.paisValue != null) {
        pais = this.paisValue;
      }

      if (nombre != "" && pais != "") {
        this.detailProducto = await this.loadProductoFilter(nombre, pais);
        this.addCotizacion(this.detailProducto);
      } else {
        this.visible = true;
      }
    },

    async loadProductos() {
      return await axios
        .get("http://localhost:9090/products")
        .then(function(response) {
          return response.data;
        })
        .catch(function(error) {
          console.log(error);
          return [];
        });
    },

    async loadData() {
      this.itemsProductos = await this.loadProductos();
      if (this.itemsProductos.length > 0) {
        this.loadPaisSelect();
        this.loadProductoSelect();
      }
    },

    loadPaisSelect() {
      let pais = [];
      this.itemsProductos.forEach(function(d) {
        let index = pais.findIndex(x => x.Country == d.Country);
        if (index === -1) {
          if (d.Country != null && d.Country != "") {
            pais.push(d.Country);
          }
        }
      });
      this.paisSelect = pais;
    },
    loadProductoSelect() {
      let producto = [];
      this.itemsProductos.forEach(function(d) {
        let index = producto.findIndex(x => x.Name == d.Name);
        if (index === -1) {
          if (d.Name != null && d.Name != "") {
            producto.push(d.Name);
          }
        }
      });
      this.productoSelect = producto;
    },

    addCotizacion(json) {
      let precio = 0;
      if (json.Type == "DOLAR") {
        precio = Number(json.Price) * parseFloat(this.dolar);
      }
      if (json.Type == "EURO") {
        precio = Number(json.Price) * parseFloat(this.euro);
      }

      if (json.length != "") {
        this.columnasCotizacion.push({
          producto: json.Name,
          pais: json.Country,
          precio: precio
        });
        this.valorTotal = precio + this.valorTotal;
        this.visible = false;
      } else {
        this.visible = true;
      }
    },

    async getDolar(moneda) {
      let fecha = new Date();
      let dia = String(fecha.getDate()).padStart(2, "0");
      let mes = String(fecha.getMonth() + 1).padStart(2, "0");
      let ano = fecha.getFullYear();
      let url = "";
      let apiKey = "<ESCRIBIR API KEY>";
      if (moneda == "euro") {
        url =
          "https://api.sbif.cl/api-sbifv3/recursos_api/euro/" +
          ano +
          "/" +
          mes +
          "/dias/" +
          dia +
          "?apikey=" +
          apiKey +
          "&formato=json";
      }
      if (moneda == "dolar") {
        url =
          "https://api.sbif.cl/api-sbifv3/recursos_api/dolar/" +
          ano +
          "/" +
          mes +
          "/dias/" +
          dia +
          "?apikey=81f49f3da8f835cf5702e64373f5c9c0aa271399&formato=json";
      }

      return await axios
        .get(url)
        .then(function(response) {
          return response.data;
        })
        .catch(function(error) {
          console.log(error);
          return [];
        });
    },
    async loadApiDolar(moneda) {
      this.valorDolar = await this.getDolar(moneda);
      if (moneda == "euro") {
        this.euro = this.valorDolar.Euros[0].Valor;
      }
      if (moneda == "dolar") {
        this.dolar = this.valorDolar.Dolares[0].Valor;
      }
    },

    borrarCotizacion() {
      this.columnasCotizacion = [];
      this.valorTotal = 0;
      this.paisValue = "";
      this.productoValue = "";
    }
  }
};
</script>
