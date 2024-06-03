<template>
    <main>
        <h1>Les catégories de produits</h1>
        <div>
        <select v-model="data.selectedCategorie" @change="handleChange">
            <option disabled value="">Veuillez choisir une catégorie</option>
            <option v-for="cate in data.listeCategories" :value="cate.code">{{ cate.libelle }}</option>
        </select>
        </div>
        <div>
            <table>
                <caption>Les produits</caption>
                <tr>
                    <th>Nom</th>
                    <th>Prix</th>
                    <th>Stock</th>
                    <th>Commandés</th>
                </tr>
                <!-- Si le tableau des produits est vide -->
    
                <tr v-if="data.listeProduitsCate.length === 0">
                    <td colspan="4">Veuillez patienter, chargement des produits...</td>
                </tr>
                <!-- Si le tableau des produits n'est pas vide -->
                <tr v-for="produit in data.listeProduitsCate" :key="produit">
                    <td>{{ produit.nom }}</td>
                    <td>{{ produit.prixUnitaire }}</td>
                    <td>{{ produit.unitesEnStock }}</td>
                    <td>{{ produit.unitesCommandees }}</td>
                    
                </tr>
            </table>
        </div>
    </main>
</template>

<script setup>
import { reactive, onMounted } from "vue";
import { doAjaxRequest, APIError } from "../api";

let data = reactive({
    // la categorie selectionné
    selectedCategorie: 1,
    // la liste des categorie
    listeCategories: [],
    // Informations de la page 
    listeProduitsCate: [],

    page: {}
});

function handleChange(){
    chargeProduitsParCate()
}
function showError(error) {
    console.log("Erreur : status %d", error.status)
    console.log(error.body);
    alert(error.message);
}

/**
 * Appel à l'API pour avoir la liste des produits par categorie
 */
function chargeProduitsParCate() {
    console.log(data.selectedCategorie);
    doAjaxRequest(`/api/categories/${data.selectedCategorie}/produits`)
        .then((json) => {
            data.listeProduitsCate = json._embedded.produits;
            data.page = json.page;
        })
        .catch(showError);
}

function chargeCategories() {
    // Appel à l'API pour avoir la liste des catégories
    // Trié par code, descendant
    // Verbe HTTP GET par défaut
    doAjaxRequest("/api/categories?sort=code,desc")
        .then((json) => {
            data.listeCategories = json._embedded.categories;
        })
        .catch(showError);
}

// A l'affichage du composant, on affiche la liste des produits de la 1ère page et des categories
onMounted(() => {
    chargeProduitsParCate(data.selectedCategorie);
    chargeCategories()
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

</style>