<template>

  <div class="todo-app">

    <h1>Ma première page Todo</h1>

    <p v-if="status === 'pending'">Chargement...</p>

    <p v-else-if="error">

      Impossible de récupérer les tâches.

    </p>
    <ul v-else>

     <li v-for="todo in todosFiltres" :key="todo.id">
        <input 
           type="checkbox"
           :checked="todo.estTerminee"
           @change="changerEtat( todo )"
        />

       <span :class="{ 'task-name': true, 'important': todo.estTerminee }">
         {{ todo.nom }}
       </span>

        <div class="task-actions">
         
          <input 
          type="text"
          v-model ="todo.nom"
          />

          <button class="btn btn-primary" @click="modifierTache(todo.id)">
            Modifier
          </button>

          <button class="btn btn-primary" @click="supprimerTache(todo.id)">
            Supprimer
           </button> 
        </div>

      </li>

    </ul>
      
      <div class="button-group">
      <button class="btn btn-filter" @click="filtre = 'toutes'">Toutes</button>
      <button class="btn btn-filter" @click="filtre = 'terminees'">Terminées</button>
      <button class="btn btn-filter" @click="filtre = 'nonTerminees'">Non terminées</button>
      </div>

      <button class="btn-primary btn-danger" @click="supprimerTerminees">Supprimer les tâches terminées</button>
        
        <div class="trés">
          <input class="place" type="texte" v-model="nouveauNom" placeholder="Écrire une tâche..." />

          <button class="btn btn-primary" @click="ajouterTache">Ajouter</button>

          <p v-if="erreurAjout" class="error-message">
            {{erreurAjout}}
          </p>
       </div>

  </div>

</template>

<script setup>
import { ref, computed } from 'vue'

const { data: todos, status, error, refresh} = await useFetch (

  'http://localhost:5003/api/Todo'

)
const nouveauNom = ref('')
const erreurAjout = ref('') 

 async function ajouterTache() {

  erreurAjout.value = ''

  if (nouveauNom.value.trim() === '') {
   erreurAjout.value = 'Le nom est obligatoire.'
   return
 }

   if (nouveauNom.value.trim().length < 3) {
    erreurAjout.value = 'Le nom doit contenir au moins 3 caractères.'
    return
   }

   if (nouveauNom.value.trim().length > 255) {
   erreurAjout.value = 'Le nom ne doit pas dépasser 255 caractères.'
   return
   }

 await $fetch(`http://localhost:5003/api/Todo`, {

    method: 'POST',

    body: {

      nom: nouveauNom.value

    }

  })

  nouveauNom.value = ''

await refresh()
}
 async function changerEtat(todo) {

  const nouvelleValeur= !todo.estTerminee


   await $fetch(`http://localhost:5003/api/Todo/${todo.id}/etat`, {

    method: 'PATCH',
    body: {
        estTerminee: nouvelleValeur
    }
    })


    todo.estTerminee = nouvelleValeur

    }

    const filtre = ref('toutes')
    const todosFiltres = computed(() => {
      if (filtre.value === 'terminees') {
         return todos.value.filter(todo => todo.estTerminee)
      }

      if (filtre.value === 'nonTerminees') {
         return todos.value.filter(todo => !todo.estTerminee)
      }

      return todos.value
       })

     async function supprimerTache(id) {

       await $fetch(`http://localhost:5003/api/Todo/${id}`, {

          method: 'DELETE',
        })
        
         await refresh()
     }

     async function supprimerTerminees() {
      const confirmer = confirm(
        'Voulez-vous vraiment Supprimer toutes les tâches terminées ?'
      )
      if (!confirmer){
        return
      }


       await $fetch(`http://localhost:5003/api/Todo/terminees`, {

          method: 'DELETE',
        })
        
         await refresh()
     }

     async function modifierTache(id) {

      const todo = todos.value.find(t => t.id === id)

       await $fetch(`http://localhost:5003/api/Todo/${id}`, {

          method: 'PUT',
          body: {
            nom: todo.nom

          }
          
        })
        
         await refresh()
      }

</script>

<style>
body {
  front-family: Arial, sans-serif;
}

h1 {
  text-align: center;
}
ul {
  list-style: none;
  padding: 0;
}
li {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 8px;
}
button {
  margin: 5px;
  padding: 7px 12px;
  cursor: pointer;
}
input[type="text"] {
  padding: 8px;
  width: 250px;
  border: 1px solid #ccc;
  boerder-radius: 6px;
}
.todo-app {
  width: 600px;
  margin: 30px auto;
}
.todo-app h1 {
  text-align: center;
  margin-bottom: 25px;
}
.button-group {
  display: flex;
  gap: 8px;
}
.btn-primary {
  background-color: #2563eb;
  color: #ffffff;
  border: 2px solid #2563eb;
}
.btn-primary:hover {
  background-color: #1d4ed8;
  border-color: #1d4ed8;

}
.btn-filter {
  background-color: transparent;
  color: #4b5563;
  border: 2px solid #d1d5db;
}
.btn-filter:hover {
  background-color: #f3f4f6;
  bordero-color: #9ca3af;
  color: #1f2937;
}
.btn-danger {
  background-color: #dc3545;
  border-color: #dc3545;
}
.btn-danger:hover {
  background-color: #bd2130;
  border-color: #bd2130;
}
.important {
  text-decoration: line-through;
  opacity: 0.5;
  color: #bd1245;
}
.task-name {
  flex: 1;
}
.task-actions {
  display: flex;
  gap: 8px;
}
.trés {
  display: flex;
  gap: 8px;
}
.place {
  flex: 1;
}
.error-message {
  color: red;
}
</style>