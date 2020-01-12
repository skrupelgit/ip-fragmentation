<template>
  <div class="container">
    <div style="display:flex; justify-content:center">
      <div v-for="(red, index) in redes" :key="index">
        <div style="    display: flex;
    flex-direction: column;
    align-items: start;">
          <div>Red {{index}}</div>
          <div>
            <img width="50px" src="https://live.staticflickr.com/6075/6125187969_b1de083e66_n.jpg" />
            <span v-show="index < redes.length-1">&lt;-----&gt;</span>
          </div>
          <div>
            <label>
              MTU
              <input v-model.number="redes[index].mtu" style="width:55px" type="number" />
            </label>
          </div>
        </div>
      </div>
    </div>
    <div>
      <button @click="newRed">Añadir red</button>
    </div>
    <div>
      Realizar ping Red 0 -> Red {{redes.length-1}}
      <br />
      <label>
        Tamaño del paquete*
        <input type="number" v-model.number="size" />
      </label>
      Protocolo
      <label>
        IpV4
      <input type="radio" value="4" v-model="protocol">

      </label>
      <label>
        IpV6
      <input type="radio" value="6" v-model="protocol">

      </label>
      <br />*Se obvia cabecera del ping (8 bytes)
      <button @click="calcular">Calcular</button>
    </div>
    <div>
      <Results v-for="(result, index) in results" :key="index" :index="index" :result="result"></Results>
    </div>
  </div>
</template>

<script>
import Results from "./Results.vue";
export default {
  name: "HelloWorld",
  components: { Results },
  data() {
    return {
      redes: [
        {
          mtu: 1500
        }
      ],

      protocol:"4",

      size: 1000,
      mtu: 1500,


      header: {
        "4":20,
        "6":40
      },
      results: []
    };
  },
  methods: {
    newRed() {
      this.redes.push({ mtu: 1500 });
      this.$forceUpdate();
    },
    fragment(prevResults, mtu) {
      let newResults = [];
      for (let prev in prevResults) {
        let prevResult = prevResults[prev];
        prevResult.size = prevResult.long - this.cabecera;
        if (prevResult.size + this.cabecera <= mtu) {
          newResults.push(prevResult);
        } else {
          let max_size = 0;
          if ((mtu - this.cabecera) % 8 === 0) {
            max_size = mtu - this.cabecera;
          } else {
            max_size = mtu - this.cabecera - ((mtu - this.cabecera) % 8);
          }
          let n_paquetes = parseInt(prevResult.size / max_size);
          let offset = max_size / 8;
          let packetLeft = this.size;
          let ite_offeset = prevResult.offset;

          for (let i = 0; i <= n_paquetes; i++) {
            let long = 0;
            let mf = 0;
            if (packetLeft >= max_size) {
              packetLeft = packetLeft - max_size;
              long = max_size + this.cabecera;
            } else {
              long = packetLeft + this.cabecera;
            }
            if (prevResult.mf || i != n_paquetes) mf = 1;
            newResults.push({
              offset: ite_offeset,
              mf: mf,
              df: 0,
              long: long
            });
            ite_offeset += offset;
          }
        }
      }

      return newResults;
    },
    getmtu(red){
      if(this.protocol==="4"){
        return this.redes[red].mtu
      }
      else{
        let minMtu=Infinity
          for(let i in this.redes ){
            if(this.redes[i].mtu<minMtu) minMtu=this.redes[i].mtu;
          }
          return minMtu;
      }
    },
    calcular() {
      this.results = [];
      for (let red in this.redes) {
        this.results[red] = [];
        let mtu=this.getmtu(red);
        if (red == 0) {

          


          this.results[red].push(
            this.fragment(
              [
                {
                  long: this.size + this.cabecera,
                  mf: 0,
                  df: 0,
                  offset: 0
                }
              ],
              mtu
            )
          );
        } else {
          this.results[red].push(
            this.fragment(this.results[red - 1][0], mtu)
          );
        }
      }

      this.$forceUpdate();
    }
  },
  computed:{
    cabecera(){
      return this.header[this.protocol]
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
