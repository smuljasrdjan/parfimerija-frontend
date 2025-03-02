<template>
  <div id="app">
    <h1>Parfimerija Sergio</h1>
    
    <!-- Forma za unos kupca -->
    <div>
      <h2>Unesi kupca</h2>
      <input v-model="newCustomer.name" placeholder="Ime" />
      <input v-model="newCustomer.surname" placeholder="Prezime" />
      <input v-model="newCustomer.dob" placeholder="Datum rođenja (YYYY-MM-DD)" />
      <input v-model="newCustomer.phone" placeholder="Broj telefona" />
      <label>
        <input type="checkbox" v-model="newCustomer.has_viber" /> Ima Viber
      </label>
      <button @click="addCustomer">Dodaj kupca</button>
    </div>
    
    <!-- Lista rođendana sa Viber linkovima -->
    <div>
      <h2>Kupci sa današnjim rođendanom (Viber)</h2>
      <button @click="fetchViberLinks">Prikaži Viber korisnike</button>
      <ul>
        <li v-for="link in viberLinks" :key="link">
          <a :href="link" target="_blank">Pošalji poruku</a>
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
        this.newCustomer = { name: "", surname: "", dob: "", phone: "", has_viber: false }; // Reset forme
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
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}
input {
  display: block;
  margin: 10px 0;
  padding: 5px;
}
button {
  padding: 10px;
  margin: 10px 0;
}
</style>