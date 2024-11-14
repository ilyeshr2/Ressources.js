<template>
  <div id="app" :class="{'dark-theme': theme === 'dark'}" class="container">
    <header>
      <h1>Gestion des Ressources</h1>
      <div class="header-controls">
        <button @click="toggleStats" class="stats-button">
          {{ showStats ? 'Masquer' : 'Voir' }} Statistiques
        </button>
        <button @click="toggleTheme">
          {{ theme === 'dark' ? 'Thème Clair' : 'Thème Sombre' }}
        </button>
      </div>
    </header>

    <section v-if="showStats" class="stats-section">
      <div class="stats-grid">
        <div class="stat-card">
          <h3>Total des Ressources</h3>
          <p>{{ ressources.length }}</p>
        </div>
        <div class="stat-card">
          <h3>Par Type</h3>
          <ul>
            <li v-for="(count, type) in resourcesByType" :key="type">
              {{ type }}: {{ count }}
            </li>
          </ul>
        </div>
        <div class="stat-card">
          <h3>Dernière Modification</h3>
          <p>{{ lastModified }}</p>
        </div>
      </div>

      <div class="graph-section">
        <h3>Graphique des Ressources</h3>
        <canvas id="resourcesChart"></canvas>
      </div>
    </section>


    <section class="form-section">
      <form @submit.prevent="ajouterOuMettreAJourRessource">
        <div class="form-group">
          <label for="nom">Nom de la Ressource:</label>
          <input v-model="nouvelleRessource.nom" type="text" id="nom" required />
          <span v-if="erreurs.nom" class="error">{{ erreurs.nom }}</span>
        </div>
        <div class="form-group">
          <label for="type">Type de Ressource:</label>
          <select v-model="nouvelleRessource.type" id="type" required>
            <option disabled value="">Sélectionnez un type</option>
            <option value="Livre">Livre</option>
            <option value="Article">Article</option>
            <option value="Vidéo">Vidéo</option>
            <option value="Podcast">Podcast</option>
            <option value="Cours">Cours</option>
          </select>
          <span v-if="erreurs.type" class="error">{{ erreurs.type }}</span>
        </div>
        <div class="form-group">
          <label for="description">Description:</label>
          <textarea v-model="nouvelleRessource.description" id="description" rows="3"></textarea>
        </div>
        <div class="form-group">
          <label for="tags">Tags (séparés par des virgules):</label>
          <input v-model="nouvelleRessource.tags" type="text" id="tags" />
        </div>
        <div class="form-group">
          <label for="priority">Priorité:</label>
          <select v-model="nouvelleRessource.priority" id="priority">
            <option value="low">Basse</option>
            <option value="medium">Moyenne</option>
            <option value="high">Haute</option>
          </select>
        </div>
        <button class="ajt-btn" type="submit">{{ modeEdition ? "Mettre à jour" : "Ajouter" }} Ressource</button>
      </form>
    </section>

    <section class="search-section">
      <div class="search-controls">
        <input v-model="filtre" type="text" placeholder="Rechercher une ressource..." class="search-input" />
        <div class="filter-options">
          <select v-model="filterType">
            <option value="">Tous les types</option>
            <option v-for="type in availableTypes" :key="type" :value="type">
              {{ type }}
            </option>
          </select>
          <select v-model="sortBy">
            <option value="name">Trier par nom</option>
            <option value="type">Trier par type</option>
            <option value="date">Trier par date</option>
            <option value="priority">Trier par priorité</option>
          </select>
        </div>
      </div>
    </section>

    <section class="resource-list">
      <h2>Liste des Ressources</h2>
      <transition-group name="list" tag="ul">
        <li v-for="(ressource, index) in ressourcesPaginees" :key="ressource.id" :class="['resource-item', `priority-${ressource.priority}`]">
          <div class="resource-content">
            <div class="resource-header">
              <h3>{{ ressource.nom }}</h3>
            </div>
            <p class="resource-description" v-if="ressource.description">
              {{ ressource.description }}
            </p>
            <div class="resource-tags" v-if="ressource.tags">
              <span v-for="tag in ressource.tags.split(',')" :key="tag.trim()" class="tag">
                {{ tag.trim() }}
              </span>
            </div>
            <div class="resource-meta">
              <span>Créé le: {{ formatDate(ressource.dateCreation) }}</span>
            </div>
            <div class="resource-footer">
              <span class="priority-badge" :class="ressource.priority">
                {{ ressource.priority }}
              </span>
              <span class="resource-type">{{ ressource.type }}</span>
            </div>
          </div>

          <div class="resource-actions">
            <button @click="editerRessource(index)" class="edit-btn">Éditer</button>
            <button @click="confirmerSuppression(index)" class="delete-btn">Supprimer</button>
            <button @click="toggleFavorite(index)" class="favorite-btn">
              {{ ressource.favorite ? '★' : '☆' }}
            </button>
          </div>
        </li>
      </transition-group>
      <div v-if="!ressourcesPaginees.length" class="no-results">
        Aucune ressource trouvée
      </div>
      <div class="pagination">
        <button @click="pagePrecedente" :disabled="pageActuelle === 1">
          Précédent
        </button>
        <span class="page-info">
          Page {{ pageActuelle }} sur {{ pagesTotales }}
        </span>
        <button @click="pageSuivante" :disabled="pageActuelle >= pagesTotales">
          Suivant
        </button>
      </div>
    </section>

    <div v-if="toastMessage" class="toast" :class="toastType">
      {{ toastMessage }}
    </div>
  </div>
</template>

<script>
import { Chart, registerables } from 'chart.js';
Chart.register(...registerables);

export default {
  data() {
    return {
      nouvelleRessource: {
        id: null,
        nom: '',
        type: '',
        description: '',
        tags: '',
        priority: 'medium',
        dateCreation: null,
        favorite: false
      },
      ressources: JSON.parse(localStorage.getItem('ressources')) || [],
      filtre: '',
      filterType: '',
      sortBy: 'name',
      modeEdition: false,
      indexEdition: null,
      erreurs: {},
      pageActuelle: 1,
      parPage: 2,
      toastMessage: '',
      toastType: 'info',
      theme: localStorage.getItem('theme') || 'light',
      showStats: true,
      selectedTags: [],
      lastModified: localStorage.getItem('lastModified') || 'Aucune modification',
      chartInstance: null
    };
  },
  computed: {
    availableTypes() {
      return [...new Set(this.ressources.map(r => r.type))];
    },
    resourcesByType() {
      return this.ressources.reduce((acc, res) => {
        acc[res.type] = (acc[res.type] || 0) + 1;
        return acc;
      }, {});
    },
    ressourcesFiltrees() {
      return this.ressources
        .filter(ressource => {
          const matchSearch = ressource.nom.toLowerCase().includes(this.filtre.toLowerCase()) ||
            ressource.type.toLowerCase().includes(this.filtre.toLowerCase()) ||
            (ressource.description && ressource.description.toLowerCase().includes(this.filtre.toLowerCase()));
          const matchType = !this.filterType || ressource.type === this.filterType;
          const matchTags = this.selectedTags.length === 0 || 
            (ressource.tags && this.selectedTags.every(tag => 
              ressource.tags.toLowerCase().includes(tag.toLowerCase())
            ));
          return matchSearch && matchType && matchTags;
        })
        .sort((a, b) => {
          switch (this.sortBy) {
          case 'name': return a.nom.localeCompare(b.nom);
          case 'type': return a.type.localeCompare(b.type);
          case 'date': return new Date(b.dateCreation) - new Date(a.dateCreation);
          case 'priority': {
            const priorityOrder = { high: 3, medium: 2, low: 1 };
            return priorityOrder[b.priority] - priorityOrder[a.priority];
          }
          default: return 0;
        }});
    },
    pagesTotales() {
      return Math.ceil(this.ressourcesFiltrees.length / this.parPage);
    },
    ressourcesPaginees() {
      const start = (this.pageActuelle - 1) * this.parPage;
      return this.ressourcesFiltrees.slice(start, start + this.parPage);
    }
  },
  watch: {
    ressources() {
      this.updateChart();
    },
    ressourcesFiltrees() {
      this.pageActuelle = 1; 
    }
  },
  methods: {
    allerALaPage(page) {
      if (page >= 1 && page <= this.pagesTotales) {
        this.pageActuelle = page;
      }
    },
    pagePrecedente() {
      this.allerALaPage(this.pageActuelle - 1);
    },
    pageSuivante() {
      this.allerALaPage(this.pageActuelle + 1);
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('fr-FR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    },
    toggleStats() {
      this.showStats = !this.showStats;
    },
    toggleTag(tag) {
      const index = this.selectedTags.indexOf(tag);
      if (index === -1) {
        this.selectedTags.push(tag);
      } else {
        this.selectedTags.splice(index, 1);
      }
    },
    toggleFavorite(index) {
      this.ressources[index].favorite = !this.ressources[index].favorite;
      this.sauvegarderRessources();
      this.afficherToast(
        `Ressource ${this.ressources[index].favorite ? 'ajoutée aux' : 'retirée des'} favoris`,
        'info'
      );
    },
    validerFormulaire() {
      this.erreurs = {};
      if (!this.nouvelleRessource.nom) this.erreurs.nom = "Le nom est requis.";
      if (!this.nouvelleRessource.type) this.erreurs.type = "Le type de ressource est requis.";
      return Object.keys(this.erreurs).length === 0;
    },
    ajouterOuMettreAJourRessource() {
      if (!this.validerFormulaire()) return;
      
      const ressource = {
        ...this.nouvelleRessource,
        id: this.modeEdition ? this.nouvelleRessource.id : Date.now(),
        dateCreation: this.modeEdition ? this.nouvelleRessource.dateCreation : new Date().toISOString()
      };

      if (this.modeEdition) {
        this.ressources.splice(this.indexEdition, 1, ressource);
        this.afficherToast("Ressource mise à jour", "success");
        this.modeEdition = false;
        this.indexEdition = null;
      } else {
        this.ressources.push(ressource);
        this.afficherToast("Ressource ajoutée", "success");
      }
      
      this.lastModified = new Date().toLocaleString();
      this.sauvegarderRessources();
      this.reinitialiserFormulaire();
      this.updateChart(); 
    },
    confirmerSuppression(index) {
      if (confirm("Êtes-vous sûr de vouloir supprimer cette ressource ?")) {
        this.supprimerRessource(index);
      }
    },
    supprimerRessource(index) {
      this.ressources.splice(index, 1);
      this.sauvegarderRessources();
      this.afficherToast("Ressource supprimée", "warning");
      this.updateChart(); 
    },
    editerRessource(index) {
      this.nouvelleRessource = { ...this.ressources[index] };
      this.modeEdition = true;
      this.indexEdition = index;
    },
    sauvegarderRessources() {
      localStorage.setItem('ressources', JSON.stringify(this.ressources));
      localStorage.setItem('lastModified', this.lastModified);
    },
    reinitialiserFormulaire() {
      this.nouvelleRessource = {
        id: null,
        nom: '',
        type: '',
        description: '',
        tags: '',
        priority: 'medium',
        dateCreation: null,
        favorite: false
      };
      this.erreurs = {};
    },
    afficherToast(message, type = 'info') {
      this.toastMessage = message;
      this.toastType = type;
      setTimeout(() => {
        this.toastMessage = '';
        this.toastType = 'info';
      }, 3000);
    },
    toggleTheme() {
      this.theme = this.theme === 'dark' ? 'light' : 'dark';
      localStorage.setItem('theme', this.theme);
    },
    initializeChart() {
      const ctx = document.getElementById('resourcesChart').getContext('2d');
      this.chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: Object.keys(this.resourcesByType),
          datasets: [{
            label: 'Nombre de Ressources',
            data: Object.values(this.resourcesByType),
            backgroundColor: 'rgba(91, 192, 190, 0.6)',
            borderColor: 'rgba(91, 192, 190, 1)',
            borderWidth: 1
          }]
        },
        options: {
          responsive: true,
          scales: {
            y: {
              beginAtZero: true
            }
          }
        }
      });
    },
    updateChart() {
      if (this.chartInstance) {
        this.chartInstance.data.labels = Object.keys(this.resourcesByType);
        this.chartInstance.data.datasets[0].data = Object.values(this.resourcesByType);
        this.chartInstance.update();
      }
    }
  },
  mounted() {
    this.initializeChart();
  }
};
</script>


<style>

body {
  
  font-family: 'Roboto', sans-serif;
  margin: 0;
  padding: 0;
  color: #2e2e2e;
  background-color: #f0f4f8;
}

.graph-section {
  margin-top: 20px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.graph-section canvas {
  width: 50% !important;
  height: 50% !important;
  max-width: 50%;
  height: auto;
}

.resource-footer {
  display: inline-flex;
  align-items: center;
  gap: 8px; 
  margin-top: 10px;
}


.container {
  max-width: 900px;
  margin: 30px auto;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  background-color: #ffffff;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 20px;
  margin-bottom: 25px;
  border-bottom: 2px solid #e0e0e0;
}

.header-controls {
  display: flex;
  gap: 10px;
}

header h1 {
  font-size: 28px;
  color: #3a506b;
  margin: 0;
}

header button {
  padding: 8px 16px;
  background-color: #3a506b;
  color: #fff;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s ease;
}

header button:hover {
  background-color: #5bc0be;
  transform: translateY(-1px);
}

.stats-section {
  margin-bottom: 30px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
}

.stat-card {
  background-color: #ffffff;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.stat-card h3 {
  margin: 0 0 10px 0;
  font-size: 16px;
  color: #3a506b;
}

.stat-card p {
  margin: 0;
  font-size: 24px;
  font-weight: bold;
  color: #5bc0be;
}

.form-section {
  background-color: #ffffff;
  padding: 25px;
  border-radius: 8px;
  margin-bottom: 25px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-size: 14px;
  color: #4a4a4a;
  font-weight: 500;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 16px;
  color: #2e2e2e;
  transition: border-color 0.3s ease;
}

.form-group textarea {
  resize: vertical;
  min-height: 80px;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #5bc0be;
  box-shadow: 0 0 0 2px rgba(91, 192, 190, 0.1);
}

.search-section {
  margin-bottom: 25px;
}

.search-controls {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
}

.search-input {
  flex: 1;
  min-width: 200px;
}

.filter-options {
  display: flex;
  gap: 10px;
}

.filter-options select {
  padding: 8px 12px;
  border-radius: 6px;
  border: 1px solid #ddd;
  background-color: white;
  cursor: pointer;
}

.tag-cloud {
  margin-bottom: 25px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 8px;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tag {
  padding: 4px 8px;
  background-color: #e9ecef;
  border-radius: 4px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.tag:hover {
  background-color: #5bc0be;
  color: white;
}

.tag.active {
  background-color: #3a506b;
  color: white;
}

.resource-list {
  background-color: #ffffff;
  padding: 25px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.resource-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 15px;
  margin-bottom: 15px;
  border-radius: 8px;
  background-color: #f8f9fa;
  transition: all 0.3s ease;
}

.resource-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.resource-content {
  flex: 1;
}

.resource-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.resource-header h3 {
  margin: 0;
  font-size: 18px;
  color: #3a506b;
}

.resource-type {
  padding: 4px 8px;
  background-color: #6884a5;
  color: white;
  border-radius: 4px;
  font-size: 12px;
}


.resource-description {
  margin: 10px 0;
  color: #666;
  font-size: 14px;
}

.resource-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 10px;
}

.resource-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
  font-size: 12px;
  color: #666;
}

.priority-badge {
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  color: white;
  text-transform: capitalize;
}

.priority-badge.high {
  background-color: #ffcfd0;
  color: #3a506b;
}

.priority-badge.medium {
  background-color: #fce8c0;
  color: #3a506b;
}

.priority-badge.low {
  background-color: #defacf;
  color: #3a506b;
}

.resource-actions {
  display: flex;
  gap: 8px;
}

.resource-actions button {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.edit-btn {
  background-color: #6884a5;
  color: white;
}

.delete-btn {
  background-color: #fc7d7f;
  color: white;
}

.favorite-btn {
  background-color: transparent;
  color: #faad14;
  font-size: 20px;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  margin-top: 25px;
}

.pagination button {
  padding: 8px 16px;
  background-color: #3a506b;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}
.ajt-btn{
  padding: 8px 16px;
  background-color: #3a506b;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.pagination button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.pagination button:hover:not(:disabled) {
  background-color: #5bc0be;
}

.page-info {
  font-size: 14px;
  color: #666;
}

.toast {
  position: fixed;
  bottom: 20px;
  right: 20px;
  padding: 12px 24px;
  border-radius: 8px;
  color: white;
  font-size: 14px;
  z-index: 1000;
  animation: slideIn 0.3s ease-out;
}

.toast.success {
  background-color: #52c41a;
}

.toast.warning {
  background-color: #faad14;
}

.toast.error {
  background-color: #ff4d4f;
}

.toast.info {
  background-color: #3a506b;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}

.dark-theme {
  background-color: #1a1a1a;
  color: #e0e0e0;
}

.dark-theme .container {
  background-color: #2d2d2d;
}

.dark-theme .form-section,
.dark-theme .resource-list,
.dark-theme .stat-card {
  background-color: #363636;
}

.dark-theme .resource-item {
  background-color: #2d2d2d;
}

.dark-theme .form-group input,
.dark-theme .form-group select,
.dark-theme .form-group textarea {
  background-color: #404040;
  color: #e0e0e0;
  border-color: #505050;
}

.dark-theme .resource-header h3 {
  color: #5bc0be;
}

.dark-theme .tag {
  background-color: #404040;
  color: #e0e0e0;
}

.dark-theme .resource-description {
  color: #b0b0b0;
}

.dark-theme .stats-section {
  background-color: #363636;
}

.dark-theme .tag-cloud {
  background-color: #363636;
}

.dark-theme .search-controls input,
.dark-theme .search-controls select {
  background-color: #404040;
  color: #e0e0e0;
  border-color: #505050;
}

</style>