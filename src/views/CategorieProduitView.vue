<template>
<main>
    <h1>Les catégories de produits</h1>
    <div>
      <select v-model="data.selectedCategorie" @change="handleChange">
        <option disabled value="">Veuillez choisir une catégorie</option>
        <option v-for="cate in data.listeCategories" :value="cate.code">{{ cate.libelle }}</option>
      </select>
      <input v-model="searchTerm" placeholder="Search for a product" />
      <div>
        <label>
          Min Price:
          <input type="number" v-model.number="priceRange.min" />
        </label>
        <label>
          Max Price:
          <input type="number" v-model.number="priceRange.max" />
        </label>
        <button @click="filterByPrice">Filter by Price</button>
        <button @click="() => sortByPrice(true)">Sort by Price (Ascending)</button>
        <button @click="() => sortByPrice(false)">Sort by Price (Descending)</button>
        <button @click="resetFilters">Reset Filters</button>
      </div>
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
          <tr v-if="isLoading">
            <td colspan="4">Loading products...</td>
          </tr>
          <tr v-else-if="data.listeProduitsCate.length === 0">
            <td colspan="4">No products found</td>
          </tr>
          <tr v-else v-for="produit in filteredProducts" :key="produit">
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
  import { reactive, onMounted, ref, computed } from "vue";
  import { doAjaxRequest, APIError } from "../api";
  
  let data = reactive({
    selectedCategorie: 1,
    listeCategories: [],
    listeProduitsCate: [],
    page: {}
  });
  
  let isLoading = ref(false);
  let searchTerm = ref('');
  
  let filteredProducts = computed(() => {
    if (!searchTerm.value) return data.listeProduitsCate;
    return data.listeProduitsCate.filter(produit => produit.nom.includes(searchTerm.value));
  });
  
  function handleChange(){
    chargeProduitsParCate()
  }
  
  function showError(error) {
    console.log("Erreur : status %d", error.status)
    console.log(error.body);
    alert(error.message);
  }
  
  function chargeProduitsParCate() {
    isLoading.value = true;
    doAjaxRequest(`/api/categories/${data.selectedCategorie}/produits`)
      .then((json) => {
        data.listeProduitsCate = json._embedded.produits;
        data.page = json.page;
      })
      .catch(showError)
      .finally(() => {
        isLoading.value = false;
      });
  }
  
  function chargeCategories() {
    doAjaxRequest("/api/categories?sort=code,desc")
      .then((json) => {
        data.listeCategories = json._embedded.categories;
      })
      .catch(showError);
  }
  
  onMounted(() => {
    chargeProduitsParCate(data.selectedCategorie);
    chargeCategories()
  });
  let priceRange = reactive({ min: 0, max: Infinity });

function filterByPrice() {
  data.listeProduitsCate = data.listeProduitsCate.filter(
    (produit) => produit.prixUnitaire >= priceRange.min && produit.prixUnitaire <= priceRange.max
  );
}

function sortByPrice(ascending = true) {
  data.listeProduitsCate.sort((a, b) => {
    if (ascending) {
      return a.prixUnitaire - b.prixUnitaire;
    } else {
      return b.prixUnitaire - a.prixUnitaire;
    }
  });
}

function resetFilters() {
  chargeProduitsParCate();
  priceRange.min = 0;
  priceRange.max = Infinity;
  return {
  data,
  isLoading,
  searchTerm,
  filteredProducts,
  handleChange,
  priceRange,
  filterByPrice,
  sortByPrice,
  resetFilters
}
};


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