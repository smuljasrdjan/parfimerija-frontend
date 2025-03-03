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
      <button @click="addCustomer">Dodaj kupca</button>
    </div>

    <!-- Lista rođendana sa Viber linkovima -->
    <div class="section">
      <h2>Kupci</h2>
      <div class="viber-buttons">
        <button @click="fetchViberLinksToday">Rođendani danas</button>
        <div class="month-selector">
          <select v-model="selectedMonth">
            <option value="01">Januar</option>
            <option value="02">Februar</option>
            <option value="03">Mart</option>
            <option value="04">April</option>
            <option value="05">Maj</option>
            <option value="06">Jun</option>
            <option value="07">Jul</option>
            <option value="08">Avgust</option>
            <option value="09">Septembar</option>
            <option value="10">Oktobar</option>
            <option value="11">Novembar</option>
            <option value="12">Decembar</option>
          </select>
          <button @click="fetchViberLinksMonth">Rođendani u mesecu</button>
        </div>
      </div>
      <ul class="viber-list">
        <li v-for="customer in viberLinks" :key="customer.phone" :class="getCustomerClass(customer)" class="viber-item">
          {{ customer.name }} {{ customer.surname }} - 
          <span class="send-link" @click="sendViberMessage(customer)">Pošalji poruku</span>
          <span v-if="!customer.noViber && !customer.sent" class="confirm-link" @click="confirmSent(customer)">Potvrdi slanje</span>
          <span v-if="customer.sent" class="sent-icon">✓</span>
          <span v-if="customer.noViber" class="no-viber">Kupac nema Viber</span>
        </li>
      </ul>
    </div>

    <!-- Dugme za podešavanje poruka -->
    <button class="settings-btn" @click="fetchMessagesAndShowModal">Podesi poruke</button>

    <!-- Modal za unos i kopiranje poruka -->
    <div v-if="showMessageModal" class="modal-overlay" @click="closeModal">
      <div class="modal" @click.stop>
        <h3>Podesi poruke</h3>
        <label>Današnji rođendani:</label>
        <textarea v-model="todayMessage" rows="3" placeholder="npr. 'Srećan rođendan, {name}!'"></textarea>
        <button @click="copyMessage(todayMessage)">Kopiraj današnju poruku</button>
        <label>Rođendani u mesecu:</label>
        <textarea v-model="monthMessage" rows="3" placeholder="npr. 'U vašem rođendanskom mesecu, {name}, iskoristite 20% popusta!'"></textarea>
        <button @click="copyMessage(monthMessage)">Kopiraj mesečnu poruku</button>
        <button @click="saveMessages">Sačuvaj i zatvori</button>
      </div>
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
        phone: ""
      },
      todayMessage: "",
      monthMessage: "",
      selectedMonth: new Date().getMonth() + 2 > 12 ? "01" : String(new Date().getMonth() + 2).padStart(2, "0"),
      viberLinks: [],
      showMessageModal: false
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
        this.newCustomer = { name: "", surname: "", dob: "", phone: "" };
      } catch (error) {
        alert("Greška: " + error);
      }
    },
    async fetchViberLinksToday() {
      try {
        const url = `https://parfimerija-backend.onrender.com/viber-links-today?message=${encodeURIComponent(this.todayMessage)}`;
        const response = await fetch(url, { signal: AbortSignal.timeout(30000) });
        if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
        this.viberLinks = await response.json();
      } catch (error) {
        if (error.name === "TimeoutError") {
          alert("Server se budi, pokušaj ponovo za trenutak.");
        } else {
          alert("Greška pri dohvatanju Viber linkova: " + error);
        }
      }
    },
    async fetchViberLinksMonth() {
      try {
        const url = `https://parfimerija-backend.onrender.com/viber-links-month?month=${this.selectedMonth}&message=${encodeURIComponent(this.monthMessage)}`;
        const response = await fetch(url, { signal: AbortSignal.timeout(30000) });
        if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
        this.viberLinks = await response.json();
      } catch (error) {
        if (error.name === "TimeoutError") {
          alert("Server se budi, pokušaj ponovo za trenutak.");
        } else {
          alert("Greška pri dohvatanju Viber linkova: " + error);
        }
      }
    },
    async sendViberMessage(customer) {
      window.location.href = customer.link;
    },
    async confirmSent(customer) {
      try {
        await fetch("https://parfimerija-backend.onrender.com/mark-sent", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ phone: customer.phone })
        });
        customer.sent = true;
        customer.noViber = false;
      } catch (error) {
        alert("Greška pri označavanju poruke kao poslate: " + error);
      }
    },
    async fetchMessagesAndShowModal() {
      try {
        const response = await fetch("https://parfimerija-backend.onrender.com/messages");
        const messages = await response.json();
        this.todayMessage = messages.today;
        this.monthMessage = messages.month;
        this.showMessageModal = true;
      } catch (error) {
        alert("Greška pri dohvatanju poruka: " + error);
      }
    },
    copyMessage(message) {
      navigator.clipboard.writeText(message)
        .then(() => {
          alert("Poruka kopirana u clipboard: " + message);
        })
        .catch(err => {
          alert("Greška pri kopiranju poruke: " + err);
        });
    },
    async saveMessages() {
      try {
        await fetch("https://parfimerija-backend.onrender.com/messages", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ today: this.todayMessage, month: this.monthMessage })
        });
        this.showMessageModal = false;
      } catch (error) {
        alert("Greška pri čuvanju poruka: " + error);
      }
    },
    getCustomerClass(customer) {
      return {
        "sent": customer.sent,
        "no-viber": customer.noViber
      };
    },
    closeModal(event) {
      if (event.target.classList.contains("modal-overlay")) {
        this.showMessageModal = false;
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
  position: relative;
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
input, textarea, select {
  display: block;
  width: 100%;
  margin: 10px 0;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}
textarea {
  resize: vertical;
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
.viber-buttons {
  display: flex;
  gap: 20px;
  align-items: center;
}
.month-selector {
  display: flex;
  gap: 10px;
  align-items: center;
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
  align-items: center;
  gap: 10px;
}
.viber-item.sent {
  background-color: #e6ffe6;
}
.viber-item.no-viber {
  background-color: #ffe6e6;
}
.send-link, .confirm-link {
  color: #0078d4;
  text-decoration: none;
  cursor: pointer;
}
.send-link:hover, .confirm-link:hover {
  text-decoration: underline;
}
.sent-icon {
  color: #4CAF50;
  font-weight: bold;
}
.no-viber {
  color: #d32f2f;
  font-style: italic;
}
.settings-btn {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background-color: #0078d4;
  padding: 10px;
  border-radius: 50%;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
  max-width: 90%;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}
.modal h3 {
  margin-top: 0;
}
.modal label {
  display: block;
  margin: 10px 0 5px;
}
.modal button {
  margin: 5px 0;
}
</style>