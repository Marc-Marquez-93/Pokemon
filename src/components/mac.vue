<script setup>
import { ref, nextTick } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { gsap } from "gsap";

let pok = ref("");
let resultado = ref(null);
let imagensrc = ref("");
let tipos = ref([]);

const coloresTipo = {
  normal: "#A8A77A",
  fire: "#EE8130",
  water: "#6390F0",
  electric: "#F7D02C",
  grass: "#7AC74C",
  ice: "#96D9D6",
  fighting: "#C22E28",
  poison: "#A33EA1",
  ground: "#E2BF65",
  flying: "#A98FF3",
  psychic: "#F95587",
  bug: "#A6B91A",
  rock: "#B6A136",
  ghost: "#735797",
  dragon: "#6F35FC",
  dark: "#705746",
  steel: "#B7B7CE",
  fairy: "#D685AD",
};

async function cargarTipos() {
  try {
    const res = await axios.get("https://pokeapi.co/api/v2/type");
    const promesas = res.data.results.map(async tipo => {
      const det = await axios.get(tipo.url);
      return {
        tipo: det.data.id,
        debilidades: det.data.damage_relations.double_damage_from.map(d => d.name)
      };
    });
    tipos.value = await Promise.all(promesas);
  } catch (err) {
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "No se pudieron cargar los tipos 😢",
    });
  }
}
cargarTipos();

function obtenerDebilidades(pokemon) {
  if (!pokemon.types) return [];
  let debs = [];
  pokemon.types.forEach(t => {
    const tipoId = parseInt(t.type.url.split("/").slice(-2)[0]);
    const tipoEncontrado = tipos.value.find(x => x.tipo === tipoId);
    if (tipoEncontrado) debs.push(...tipoEncontrado.debilidades);
  });
  return [...new Set(debs)];
}

function obtenerGradiente(pokemon) {
  if (!pokemon.types) return "#fff";
  const colores = pokemon.types.map(t => coloresTipo[t.type.name]);
  return colores.length === 1
    ? colores[0]
    : `linear-gradient(135deg, ${colores.join(", ")})`;
}

// 🟡 Nueva función: recargar la página y luego pintar
function buscar() {
  if (!pok.value.trim()) {
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Por favor ingresa el nombre o código de un Pokémon válido!",
    });
    return;
  }

  // Guardar el valor del input antes de recargar
  localStorage.setItem("ultimoPokemon", pok.value);
  location.reload();
}

// 🟢 Cuando la página se recarga, ejecuta la búsqueda automáticamente
window.addEventListener("load", async () => {
  const ultimo = localStorage.getItem("ultimoPokemon");
  if (!ultimo) return;

  pok.value = ultimo;
  localStorage.removeItem("ultimoPokemon");

  try {
    const res = await axios.get(`https://pokeapi.co/api/v2/pokemon/${pok.value.toLowerCase()}`);
    resultado.value = res.data;

    imagensrc.value =
      resultado.value.sprites.other.dream_world.front_default ||
      resultado.value.sprites.other["official-artwork"].front_default ||
      resultado.value.sprites.front_default;

    await nextTick();

    const tl = gsap.timeline();
    tl.from("#card", { opacity: 0, y: -50, duration: 0.6, ease: "power2.out" });
    tl.from("#nombre h2", { opacity: 0, y: -20, duration: 0.5, ease: "power2.out" }, "-=0.3");
    tl.from("#nombre .idd", { opacity: 0, y: 20, duration: 0.5, ease: "power2.out" }, "-=0.4");
    tl.to("#nombre img", {
      x: 20,
      y: 10,
      rotation: 6,
      duration: 1.5,
      yoyo: true,
      repeat: -1,
      ease: "sine.inOut"
    }, "-=0.3");

    const elementosPulso = [".es1", "#datos .data"];
    tl.to(elementosPulso, {
      scale: 1.05,
      duration: 0.8,
      yoyo: true,
      repeat: -1,
      ease: "sine.inOut",
      stagger: 0.2
    }, "-=1");
  } catch (err) {
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Pokémon no encontrado 😢",
    });
    pok.value = "";
    resultado.value = null;
    imagensrc.value = "";
  }
});
</script>




<template>
  <div id="container">
    <div id="tit">
      <img id="titulo"
        src="https://static.vecteezy.com/system/resources/previews/027/127/591/non_2x/pokemon-logo-pokemon-icon-transparent-free-png.png"
        alt="Pokemon" />
    </div>
    <div class="div">
      <input placeholder="escriba el pokemon (codigo o nombre)" v-model="pok" id="input" />
      <button @click="buscar" class="bt">buscar</button>
    </div>
    <div id="card" v-if="resultado && resultado" :style="{ background: obtenerGradiente(resultado) }">
      <div id="estadisticas">
        <div class="es1">
          <h3>Estadísticas</h3>
          <div v-for="stat in resultado.stats" :key="stat.stat.name">
            <p>
              {{ stat.stat.name.toUpperCase() }}:
              {{ `${stat.base_stat}/${255}` }}
            </p>
            <div class="barra">
              <div class="progreso" :style="{
                width: (stat.base_stat / 255) * 100 + '%',
                background: obtenerGradiente(resultado),
              }"></div>
            </div>
          </div>
        </div>
      </div>

      <div id="nombre">
        <div style="position: relative">
          <h2 style="position: relative; top: 50%">
            {{ resultado.name.toUpperCase() }}
          </h2>
        </div>
        <div>
          <img :src="imagensrc" alt="imagen" />
        </div>
        <div>
          <span class="idd"
            :style="{ backgroundColor: resultado.types ? coloresTipo[resultado.types[0].type.name] : '#fff' }">
            #{{ resultado.id }}
          </span>
        </div>
      </div>

      <div id="datos">
        <div class="data">
          <h3>altura: {{ `${resultado.height} metros` }}</h3>
          <h3>peso: {{ ` ${resultado.weight} kilogramos ` }}</h3>
        </div>
        <div class="data">
          <h3>Tipos:</h3>
          <span v-for="tipo in resultado.types" :key="tipo.type.name"
            :style="{ backgroundColor: coloresTipo[tipo.type.name] }" class="tipo">
            {{ tipo.type.name.toUpperCase() }}
          </span>
        </div>

        <div class="data">
          <h3>Debilidades:</h3>
          <div>
            <span v-for="d in obtenerDebilidades(resultado)" :key="d" :style="{ backgroundColor: coloresTipo[d] }"
              class="tipo">
              {{ d.toUpperCase() }}
            </span>
          </div>
        </div>
      </div>
    </div>

    <div v-else></div>
  </div>
</template>


<style scoped>
.idd {
  color: white;
  padding: 5px 10px;
  border-radius: 12px;
  font-weight: bold;
  display: inline-block;
  margin-top: 5px;
  font-size: 25px;
}

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

#card {
  margin-top: 20px;
  min-width: 300px;
  width: 80%;
  min-height: 450px;
  border: solid 2px #000000;
  border-radius: 8px;
  background-color: #ffffff;
  display: grid;
  justify-items: center;
  grid-template-columns: 2fr 1fr 2fr;
  gap: 10px;
  cursor:pointer;
}

#nombre {
  display: grid;
  justify-items: center;
  position: relative;
}

#nombre img {
  width: 200px;
  height: 200px;
}

#estadisticas {
  display: grid;
  justify-items: center;
  align-items: center;
}

.es1 {
  padding: 10px;
  border: #000000 solid 2px;
  border-radius: 8px;
  background-color: #ffcb05;
  min-width: 290px;
  width: 90%;
  gap: 5px;
  cursor: pointer;
}

.barra {
  height: 15px;
  width: 100%;
  border-radius: 12px;
  background-color: #ffcb05;
  border: solid 1px #000000;
  margin-bottom: 10px;
  overflow: hidden;
}

#datos {
  margin: 25px 0;
  display: grid;
  justify-items: center;
  align-items: center;
}

.data {
  border: #000000 solid 1px;
  border-radius: 8px;
  background-color: #ffcb05;
  min-width: 290px;
  width: 90%;
  gap: 5px;
  cursor: pointer;
  padding: 10px;
  text-align: center;
  height: 60%;
}

.progreso {
  height: 100%;
  background-color: #3b4cca;
  transition: width 1s ease;
}

.tipo {
  padding: 5px 10px;
  border-radius: 12px;
  color: white;
  margin: 5px;
  font-weight: bold;
  display: inline-block;
  cursor: pointer;
}

/* 📱 Responsividad total desde 1000px hacia abajo */
@media (max-width: 1000px) {
  body {
    margin: 0;
    padding: 0;
    background-color: rgb(235, 238, 241);
    /* mismo fondo */
    width: 100%;
    min-height: 100vh;
  }

  #container {
    width: 100%;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    background-color: rgb(235, 238, 241);
    /* mantiene el color */
    padding: 20px;
  }

  /* Card en una sola columna */
  #card {
    grid-template-columns: 1fr;
    width: 95%;
    min-height: auto;
    padding: 15px;
  }

  #estadisticas,
  #nombre,
  #datos {
    width: 100%;
    margin-bottom: 20px;
  }

  #nombre img {
    width: 160px;
    height: 160px;
  }

  /* Cajas más compactas */
  .es1,
  .data {
    max-width: 250px;
    width: 100%;
    padding: 8px;
    margin: 10px 0;
    background-color: #f2f2f2;
    border: 1px solid #000;
    border-radius: 8px;
    text-align: center;
  }

  /* Input y botón uno debajo del otro */
  .div {
    flex-direction: column;
    align-items: center;
    gap: 15px;
    width: 100%;
  }

  #input {
    width: 90%;
    font-size: 18px;
    /* legible */
  }

  .bt {
    width: 90%;
    font-size: 18px;
  }

  /* Texto legible */
  h2,
  h3,
  p {
    font-size: 18px;
  }

  /* Spans uno debajo del otro */
  .tipo {
    display: block;
    margin: 5px auto;
    width: fit-content;
  }
}
</style>