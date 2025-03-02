<template>
  <div id="app">
    <h1>Parfimerija App</h1>

    <!-- Forma za unos kupca -->
    <div class="section">
      <h2>Unesi kupca</h2>
      <input v-model="newCustomer.name" placeholder="Ime" />
      <input v-model="newCustomer.surname" placeholder="Prezime" />
      <input v-model="newCustomer.dob" placeholder="Datum rođenja (YYYY-MM-DD)" type="date" />
      <input v-model="newCustomer.phone" placeholder="Broj telefona" />
      <label>
        <input type="checkbox" v-model="newCustomer.has_viber" /> Ima Viber
      </label>
      <button @click="addCustomer">Dodaj kupca</button>
    </div>

    <!-- Lista rođendana sa Viber linkovima -->
    <div class="section">
      <h2>Kupci sa današnjim rođendanom (Viber)</h2>
      <button @click="fetchViberLinks">Prikaži Viber korisnike</button>
      <ul class="viber-list">
        <li v-for="customer in viberLinks" :key="customer.link" class="viber-item">
          {{ customer.name }} {{ customer.surname }} - 
          <a :href="customer.link" target="_blank">Pošalji poruku</a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newCustomer: {
        name: "",
        surname: "",
        dob: "",
        phone: "",
        has_viber: false
      },
      viberLinks: []
    };
  },
  methods: {
    async addCustomer() {
      try {
        const response = await fetch("https://parfimerija-backend.onrender.com/add-customer", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.newCustomer)
        });
        const result = await response.json();
        alert(result.message);
        this.newCustomer = { name: "", surname: "", dob: "", phone: "", has_viber: false };
      } catch (error) {
        alert("Greška: " + error);
      }
    },
    async fetchViberLinks() {
      try {
        const response = await fetch("https://parfimerija-backend.onrender.com/viber-links");
        this.viberLinks = await response.json();
      } catch (error) {
        alert("Greška pri dohvatanju Viber linkova: " + error);
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}
h1 {
  color: #333;
  text-align: center;
}
.section {
  margin: 20px 0;
  padding: 15px;
  background-color: #fff;
  border-radius: 5px;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.05);
}
input {
  display: block;
  width: 100%;
  margin: 10px 0;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
button {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin: 10px 0;
}
button:hover {
  background-color: #45a049;
}
.viber-list {
  list-style: none;
  padding: 0;
}
.viber-item {
  padding: 10px;
  margin: 5px 0;
  background-color: #f1f1f1;
  border-radius: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.viber-item a {
  color: #0078d4;
  text-decoration: none;
}
.viber-item a:hover {
  text-decoration: underline;
}
</style>