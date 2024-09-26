<template>
  <div style="display: flex; flex-direction: column; align-items: flex-end;">
    <button id="myBtn1" class="button_default" @click="createEditBook(null)">Adicionar</button>
    <table id="table_default">
      <tr>
        <th>Id</th>
        <th>Nome</th>
        <th>Autor</th>
        <th>Ações</th>
      </tr>
      <tr v-for="dado in dados_table">
        <td>{{ dado.id }}</td>
        <td>{{ dado.name }}</td>
        <td>{{ dado.author }}</td>
        <td>
          <button id="myBtn" class="button_default" @click="createEditBook(dado)">Editar</button>
          <button class="button_default" @click="deleteBook(dado.id)">Excluir</button>
        </td>
      </tr>
    </table>
  </div>
  
  <!-- The Modal -->
  <div id="myModal" class="modal">

    <!-- Modal content -->
    <div class="modal-content">
      <span class="close">&times;</span>
			<form class="form" id="form-novo-produto" ref="form" @submit.stop.prevent="createEditBookApi">
        <input style="display: none;" type="text" name="id" id="id" v-model="id" >
				<div class="col-12">
					<label>
						<p>Nome</p>
						<input type="text" name="nome" id="name" v-model="name" required>
						<p class="invalido" v-if="validacao.name">Por favor digite o nome do livro</p>
					</label>
				</div>
				<div class="col-12">
					<label>
						<p>Autor</p>
						<input type="text" name="autor" id="author" v-model="author" required>
						<p class="invalido" v-if="validacao.author">Por favor digite o autor do livro</p>
					</label>
				</div>
        <div class="col-12">
          <button type="submit" class="button_default">Salvar</button>
        </div>
			</form>
    </div>

  </div>
</template>

<script>
  import axios from 'axios';
  import Swal from 'sweetalert2'

	export default {
		name: "TheWelcome",
		data() {
			return {
        verificacao: true,
				dados: [],
				dados_table: [],
        livro: {},
        id: null,
        name: '',
        author: '',
        validacao: {
          name : false,
          author: false,
        },
			}
		},
    methods: {
      async init() {
        try { 
          const books = await axios.get('http://localhost:3000/api/books');
          this.dados_table = books.data;
          this.id = null
          this.name = ''
          this.author = ''
          this.validacao = {
            name : false,
            author: false,
          }
        } catch (err) {
          console.log(err);
          Swal.fire({
            title: "Falha!",
            text: "Algo deu errado!",
            icon: "error"
          });
        }
        this.verificacao = false;
      },
      async createEditBook(dado) {
        // Get the modal
        var modal = document.getElementById("myModal");

        // Get the <span> element that closes the modal
        var span = document.getElementsByClassName("close")[0];

        // When the user clicks on <span> (x), close the modal
        span.onclick = function() {
          modal.style.display = "none";
        }
        this.id = null
        this.name = ''
        this.author = ''

        if(dado != null){
          this.id = dado.id;
          this.name = dado.name;
          this.author = dado.author;
        }
        modal.style.display = "block";

        // When the user clicks anywhere outside of the modal, close it
        window.onclick = function(event) {
          if (event.target == modal) {
            modal.style.display = "none";
          }
        }
      },
			async checkFormValidity() {
				this.validacao.name = (this.name == '') ? true : false
				this.validacao.author = (this.author == '') ? true : false
				const valid = this.validacao.nome || this.validacao.cnpj
				return !valid
			},
      async deleteBook(id) {
        try {
          const books = await axios.delete('http://localhost:3000/api/books/'+id);
          this.dados = books.data;
          this.init()

          Swal.fire({
            title: "Sucesso!",
            text: "Livro deletado com sucesso!",
            icon: "success"
          });
        } catch (err) {
          console.log(err);
          Swal.fire({
            title: "Falha!",
            text: "Algo deu errado!",
            icon: "error"
          });
        }
      },
      async createEditBookApi() {
        if (!this.checkFormValidity()) {
          return
        }
        try {
          const books = await axios({
            method: ( this.id == null ? 'post' : 'put'),
            url: 'http://localhost:3000/api/books' + ( this.id == null ? '' : '/' + this.id ),
            headers: {}, 
            data: {
              name: this.name,
              author: this.author,
            }
          });
          this.dados = books.data;
          this.init()
          document.getElementById("myModal").style.display = "none";

          Swal.fire({
            title: "Sucesso!",
            text: "Livro " + ( this.id == null ? 'criado' : 'editado' ) + " com sucesso!",
            icon: "success"
          });
        } catch (err) {
          console.log(err);
          Swal.fire({
            title: "Falha!",
            text: "Algo deu errado!",
            icon: "error"
          });
        }
      },
    },
    mounted: async function() {
      if(this.verificacao){
        this.init();
      }
    },
	}
</script>


<style>
#table_default {
  font-family: Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

#table_default td, #table_default th {
  border: 1px solid #ddd;
  padding: 8px;
}

#table_default tr:nth-child(even){background-color: gray;}

#table_default th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #04AA6D;
  /* color: white; */
}

tr{
  color: white;
}

.button_default {
  background-color: #04AA6D;
  border: none;
  color: white;
  padding: 10px 15px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}

.button_default:hover {background-color: darkgreen}

/* The Modal (background) */
.modal {
  display: none; /* Hidden by default */
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  padding-top: 100px; /* Location of the box */
  left: 0;
  top: 0;
  width: 100%; /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: rgb(0,0,0); /* Fallback color */
  background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
  color: white;
}

/* Modal Content */
.modal-content {
  background-color: black;
  margin: auto;
  padding: 20px;
  border: 1px solid #888;
  width: 50%;
}

/* The Close Button */
.close {
  color: #aaaaaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: white;
  text-decoration: none;
  cursor: pointer;
}
</style>