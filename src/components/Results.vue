<template>
  <div class="row">
    <div class="col-3">Red {{index}}</div>
    <div class="col-9">
      <div v-for="(packets, i) in result[0]" :key="i">
        <table class="table table-sm">
          <thead>
            <tr>
              <th colspan="4">Paquete {{i}}</th>
            </tr>
            <tr>
              <th>Long</th>
              <th>MF</th>
              <th>DF</th>
              <th>Offset</th>
            </tr>
          </thead>

          <tr>
            <td>{{packets.long}}</td>
            <td>{{packets.mf}}</td>
            <td>{{packets.df}}</td>
            <td>{{packets.offset}}</td>
          </tr>
        </table>
        <hr />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Results",
  props: ["result", "index"]
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
