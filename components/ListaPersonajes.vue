<template>
	<div>
		<div class="rowsCont">
			<label for="rows-input">Número de registros:      </label>
			<b-form-input id="rows-input" v-model="perPage" type="number" value="5" min="1" max="20"></b-form-input>
		</div>
		<b-table striped hover bordered light :fields="columnas" :items="datos" :current-page="currentPage" @input="emiteNewCount" :per-page="perPage">
			<template #cell(imagenPersonaje)="data">
				<b-img v-if="data.item.pokemon_v2_pokemonsprites_aggregate.nodes[0].sprites!=null" :src="`${data.item.pokemon_v2_pokemonsprites_aggregate.nodes[0].sprites}`" />
				<b-img v-else :src="'https://media.istockphoto.com/id/1055079680/vector/black-linear-photo-camera-like-no-image-available.jpg?s=612x612&w=0&k=20&c=P1DebpeMIAtXj_ZbVsKVvg-duuL0v9DlrOZUvPG6UJk='"/>
			</template>
			<template #cell(height)="data">
				{{
					`${data.item.height/10}`
				}}
			</template>
			<template #cell(weight)="data">
				{{
					`${data.item.weight/10}`
				}}
			</template>
			<template #cell(pokemon_v2_pokemontypes_aggregate)="data">
				<ul class="tiposPokemon" v-for="(type, id) in data.item.pokemon_v2_pokemontypes_aggregate.nodes" :key="id">
					<li >
						{{type.pokemon_v2_type.name}}
					</li>
				</ul>

			</template>
			<template #cell(id)="data">
				<b-button class="botonRosa" @click="send(data.item.id)">Info</b-button>
			</template>
		</b-table>
		<div>
			<b-pagination v-model="currentPage" :current-page="currentPage" pills size="lg" align="center" :total-rows="rows" :per-page="perPage"></b-pagination>
		</div>
	</div>
</template>

<script>
export default {
	name: 'ListaPersonajes',
	props: {
		filterString: String,
		searchString: String
	},
	data() {
		return {
			datos: [],
			rows: 20,
      currentPage: 1,
			perPage: 5,
			count: null,
			copyFiltro: " ",
			columnas: [
        {
          label: "Pokemon",
          key: "imagenPersonaje"
        },
        {
          label: "Nombre",
          key: "name"
        },
        {
          label: "Altura  [m]",
          key: "height"
        },
        {
          label: "Peso [kg]",
          key: "weight"
        },
        {
          label: "Tipo",
          key: "pokemon_v2_pokemontypes_aggregate"
        },
        {
          label: "See more",
          key: "id"
        },
      ]
		}
	},
	mounted() {
		// console.log("Vamos a checar la página");
		this.checkPage();
		this.getData();
	},
	computed: {
		getTableData(){
			console.log("EJECUTANDO COMPUTED: "+this.currentPage);
			this.scrollToTop();
			console.log("los filtros actuales son: "+this.filterString);
			return () => this.getData();
		},
		actualiza(){
			// console.log("EJECUTANDO COMPUTED2: "+this.copyFilter);
			this.scrollToTop();
			return () => this.getTableData();
		}
	},
	methods: {
		async getData() {
			var queryVar = `
				query Pokemon_v2_pokemon {
					pokemon_v2_pokemon  {
							height
							id
							name
							weight
							pokemon_v2_pokemonsprites_aggregate {
									nodes {
											sprites(path: "other.official-artwork.front_default")
									}
							}
							pokemon_v2_pokemontypes_aggregate {
									nodes {
											pokemon_v2_type {
													name
											}
									}
							}
					}
					pokemon_v2_pokemon_aggregate {
							aggregate {
									count
							}
					}
			}
			`;
			console.log(queryVar);
			const response = await this.$axios.$post('https://beta.pokeapi.co/graphql/v1beta', { query: queryVar });
			this.datos = response.data.pokemon_v2_pokemon;

			this.rows = response.data.pokemon_v2_pokemon_aggregate.aggregate.count;
			return response.data.pokemon_v2_pokemon
		},
		emiteNewCount() {
			this.$emit('cambia-count', this.rows);
			// console.log("El valor emitido: "+this.rows);
		},
		scrollToTop() {
      // Hacer scroll suave al principio de la página
      if (process.client) {
        window.scrollTo({
          top: 0,
          behavior: 'smooth'
        });
      }
		},
		send(idValue) {
			this.$router.push({ name: 'about', params: { id: idValue, page: this.currentPage, filtros: this.filterString } });
		},
		checkPage(){
			// console.log("Estamos checando la pagina: "+this.$route.params.page);
			if(this.$route.params.page){
				// console.log("La página en la que estaba es: "+this.$route.params.page);
				this.currentPage = this.$route.params.page;
			}
			if(this.$route.params.filtros){
				// console.log("La página en la que estaba es: "+this.$route.params.page);
				this.copyFiltro = this.$route.params.filtros;
			}
		}
	},
	watch: {
		filterString: function() {
			// console.log("CAMBIA FILTER STRING: "+ this.filterString);
			this.copyFiltro = this.filterString;
			this.currentPage = 0;
			this.currentPage = 1;
			// this.copyFilter = this.filterString;
			// console.log("COPY STRING: "+ this.copyFilter);
			this.getTableData();
		}
		// ,
		// searchString: function() {
		// 	console.log("CAMBIA SEARCH STRING: "+ this.searchString);
		// 	// this.currentPage = 0;
		// 	// this.currentPage = 1;
		// 	// this.copyFilter = this.filterString;
		// 	// console.log("COPY STRING: "+ this.copyFilter);
		// 	// this.getTableData();

		// 	// Código para buscar con searchString
		// }
	}
}
</script>


<style>
img {
  width: 120px; 
  height: auto; 
}

ul {
  list-style-type: none;
	display: flex;
	justify-content: center;
	vertical-align: middle;
	padding: 0%;
}

.tiposPokemon li {
	background-color: #fdb4bf7c;
	padding: 7px;
	padding-bottom: 10px;
	align-content: center;
	justify-content: center;
	vertical-align: middle;
	width: 70%;
	border-radius: 15px;
}

.b-table tr {
  text-align: center; /* Centrado horizontal */
  vertical-align: middle; /* Centrado vertical */
}

.titulo {
  color: rgb(251, 149, 166);
  font-weight: bold;
  padding: 10px;
  margin: 20px;
  background: rgb(250, 250, 250);
}

.count {
  background-color: rgb(224, 224, 224);
  padding: 10px;
  margin: 20px;
  font-weight: bold;
}

.botonRosa {
  background : #FA8072 !important;
  border: solid 1px #fdb4bf !important;
  height: 40px;
}

.botonRosa:hover {
  background : #FA8072 !important;
  border: solid 1px #fdb4bf !important;
}

.b-pagination button { /* estilos para todos los botones */
  background-color: #f8f9fa; /* Color de fondo de los botones */
  border-color: #fdb4bf; /* Color del borde del botón activo */
  color: #e57d90; /* Color del texto de los botones */
}

.b-pagination button:hover {
  color: #ffffff; 
  border-color: #fdb4bf; 
  background-color: #fdb4bf; /* Cambiar color de fondo en hover */
  font-weight: bold;
}

.page-item.active .page-link { /* el boton de la página actual */
    color: #fff;
    background-color: #FA8072;
    border-color: #e57d90;
    font-weight: bold;
}

.page-link.active, .active > .page-link {
    border-color: #ed3591;
    background-color: #ed3591; /* Color de fondo del botón activo */
    color: white;
}

.rowsCont {
	background-color: #87CEFA;
	padding: 15px;
	display: flex;
  justify-content: right; 
	align-items: center;
}

.rowsCont input {
	width: 150px;
	vertical-align: middle;
	margin-left: 20px;
}

.rowsCont label {
	vertical-align: middle;
	color: whitesmoke;
	font-weight: bold;
	font-size: 20px;
}

</style>