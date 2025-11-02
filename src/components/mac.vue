<script setup>
import { ref } from "vue";
import axios from "axios";
import Swal from "sweetalert2";

let pok = ref("".toLowerCase());
let resultado = ref(null);

function buscar() {
  if (pok.value == "") {
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Por favor ingresa el nombre o codigo de un Pokémon!",
    });
    return;
  } 
    let config = {
      method: "get",
      maxBodyLength: Infinity,
      url: `https://pokeapi.co/api/v2/pokemon/${pok.value}`,
      headers: {},
    };

    axios
      .request(config)
      .then((response) => {
        resultado.value = response.data;
        console.log(resultado.value);
      })
      .catch((error) => {
        console.log("Pokémon no encontrado 😢", error);
        Swal.fire({
            icon: "error",
            title: "Oops...",
            text: "Pokémon no encontrado 😢",
        })
        pok.value = "";
      });
  }

</script>


<template>
  <div id="container">
    <div id="tit">
      <img
        id="titulo"
        src="https://static.vecteezy.com/system/resources/previews/027/127/591/non_2x/pokemon-logo-pokemon-icon-transparent-free-png.png"
        alt="Pokemon"
      />
    </div>
    <div class="div">
      <input
        placeholder="escriba el pokemon (codigo o nombre)"
        v-model="pok"
        id="input"
      />
      <button @click="buscar" class="bt">buscar</button>
    </div>
  </div>
</template> 


<style scoped>
#container {
  display: grid;
  justify-items: center;
  background-color: rgb(235, 238, 241);
  min-height: 100vh;
  align-content: flex-start;
}
#titulo {
  width: 400px;
  height: 200px;
  display: grid;
  justify-self: center;
}
#input {
  width: 400px;
  height: 30px;
  font-size: 20px;
  text-align: center;
  border: solid 2px #000000;
  border-radius: 8px;
  background-color: #ffcb05;
}

#input:focus {
  outline: none;
  border: solid 2px #3b4cca;
}

#input::placeholder {
  color: #3b4cca;
  opacity: 1;
}

.div {
  display: flex;
  justify-items: center;
  gap: 10px;
}
.bt {
  width: 100px;
  height: 36px;
  font-size: 20px;
  cursor: pointer;
  border: solid 2px #000000;
  border-radius: 8px;
  background-color: #3b4cca;
  color: #ffcb05;
  transition: transform 0.5s ease;
}
.bt:hover {
  background-color: #ffcb05;
  color: #3b4cca;
  border: solid 2px #3b4cca;
  transform: scale(1.05);
}
.bt:focus {
  outline: none;
  background-color: #cc0000;
}
</style>