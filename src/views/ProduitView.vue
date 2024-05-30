<template>
    <main>
        <div>
            <table>
                <caption>Les produits - page {{ data.page.number + 1 }} / {{ data.page.totalPages }} </caption>
                <tr>
                    <th>Nom</th>
                    <th>Prix</th>
                    <th>Stock</th>
                    <th>Commandés</th>
                </tr>
                <!-- Si le tableau des produits est vide -->
                <tr v-if="data.listeProduits.length === 0">
                    <td colspan="4">Veuillez patienter, chargement des produits...</td>
                </tr>
                <!-- Si le tableau des produits n'est pas vide -->
                <tr v-for="produit in data.listeProduits" :key="produit">
                    <td>{{ produit.nom }}</td>
                    <td>{{ produit.prixUnitaire }}</td>
                    <td>{{ produit.unitesEnStock }}</td>
                    <td>{{ produit.unitesCommandees }}</td>
                </tr>
            </table>
            <div class="pagination">
                <button @click="chargeProduits(0)">
                    Début
                </button>
                <button @click="data.page.number + 1 > 1 ? chargeProduits(data.page.number - 1) : ''">
                    Précédent
                </button>
                <button @click="data.page.number + 1 < data.page.totalPages ? chargeProduits(data.page.number + 1) : ''">
                    Suivant
                </button>
                <button @click="chargeProduits(data.page.totalPages - 1)">
                    Fin
                </button>
            </div>
        </div>
    </main>
</template>

<script setup>
import { reactive, onMounted } from "vue";
import { doAjaxRequest, APIError } from "../api";

let data = reactive({
    // La liste des produits affichée sous forme de table
    listeProduits: [],
    // Informations de la page 
    page: {}
});

function showError(error) {
    console.log("Erreur : status %d", error.status)
    console.log(error.body);
    alert(error.message);
}

/**
 * Appel à l'API pour avoir la liste des produits
 * Enregistre les produits dans un tableau d'une page passée en paramètre
 * Trié par nom, ascendant
 * @param {number} nPage Numéro de la page 
 */
function chargeProduits(nPage) {
    doAjaxRequest(`/api/produits?page=${nPage}&size=5&sort=nom,asc`)
        .then((json) => {
            data.listeProduits = json._embedded.produits;
            data.page = json.page;
        })
        .catch(showError);
}

// A l'affichage du composant, on affiche la liste des produits de la 1ère page
onMounted(() => {
    chargeProduits(0);
});
</script>


<style scoped>
td,
th {
    border: 1px solid #ddd;
    padding: 8px;
}

th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #232623;
    color: rgb(255, 255, 255);
}

table {
  margin-top: 20px;
  border-collapse: collapse;
  width: 100%;
}
.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.pagination button {
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 16px;
  margin: 0 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.pagination button:hover {
  background-color: #45a049;
}

.pagination button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.pagination button:first-child,
.pagination button:last-child {
  background-color: #555555;
}

.pagination button:first-child:hover,
.pagination button:last-child:hover {
  background-color: #4d4d4d;
}

</style>