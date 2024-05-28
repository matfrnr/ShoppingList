<template>
  <div class="shopping-list-background">
    <div class="bubbles">
      <div v-for="n in 50" :key="n" class="bubble"></div>
    </div>
    <div class="shopping-list-container">
      <h1 class="title">Liste de course</h1>
      <div class="input-group">
        <input v-model="newItem.name" @input="filterInput" @keyup.enter="addItem" placeholder="Ajouter un produit" maxlength="30" />
        <input type="number" v-model="newItem.quantity" min="1" max="10" placeholder="Qty" class="quantity-input" />
        <button @click="addItem">Add</button>
      </div>
      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>

      <div class="font-selector">
        <label for="fontSelector">Police : </label>
        <select id="fontSelector" v-model="selectedFont" @change="changeFont" class="font-select">
          <option value="'Bastian Script', cursive">Bastian Script</option>
          <option value="'OpenDyslexic', sans-serif">Open Dyslexic</option>
        </select>
      </div>

      <ul class="shopping-list" :style="{ fontFamily: selectedFont }">
        <li v-for="(item, index) in items" :key="index" :class="{ purchased: item.purchased }">
          <span @click="togglePurchased(index)">
            {{ item.name }} - {{ item.quantity }}
          </span>
          <div class="icons">
            <vue-feather type="edit-2" @click="openEditModal(index)" class="icon edit-icon"></vue-feather>
            <vue-feather type="trash" @click="removeItem(index)" class="icon delete-icon"></vue-feather>
          </div>
        </li>
      </ul>

      <button @click="clearAll" class="clear-button">Tout supprimer</button>
    </div>

    <div v-if="showEditModal" class="modal-overlay" @click.self="closeEditModal">
      <div class="modal-content">
        <h2>Edit Item</h2>
        <label for="editName">Name:</label>
        <input id="editName" v-model="editItemData.name" @input="filterEditInput" maxlength="30" />
        <label for="editQuantity">Quantity:</label>
        <input type="number" id="editQuantity" v-model="editItemData.quantity" min="1" max="10" />
        <button @click="saveEdit">Save</button>
        <button @click="closeEditModal">Cancel</button>
        <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch } from 'vue';
import FeatherIcon from 'vue-feather';

export default {
  components: {
    FeatherIcon
  },
  setup() {
    const newItem = ref({ name: '', quantity: 1 });
    const items = ref(JSON.parse(localStorage.getItem('shoppingListItems')) || []);
    const selectedFont = ref("'OpenDyslexic', sans-serif");
    const showEditModal = ref(false);
    const editItemData = ref({ name: '', quantity: 1 });
    const currentEditIndex = ref(null);
    const errorMessage = ref('');

    const saveItems = () => {
      localStorage.setItem('shoppingListItems', JSON.stringify(items.value));
    };

    const addItem = () => {
      if (newItem.value.name.trim() === '') {
        errorMessage.value = 'Saisir un produit';
        return;
      }
      if (newItem.value.quantity > 10) {
        errorMessage.value = 'Quantité max : 10';
        return;
      }
      if (newItem.value.name.trim() !== '' && newItem.value.quantity > 0 && newItem.value.quantity <= 10 && newItem.value.name.length <= 30) {
        items.value.push({ ...newItem.value, purchased: false });
        newItem.value.name = '';
        newItem.value.quantity = 1;
        errorMessage.value = '';
        saveItems();
      }
    };

    const removeItem = (index) => {
      items.value.splice(index, 1);
      saveItems();
    };

    const togglePurchased = (index) => {
      items.value[index].purchased = !items.value[index].purchased;
      saveItems();
    };

    const openEditModal = (index) => {
      currentEditIndex.value = index;
      editItemData.value = { ...items.value[index] };
      showEditModal.value = true;
      errorMessage.value = '';
    };

    const closeEditModal = () => {
      showEditModal.value = false;
    };

    const saveEdit = () => {
      if (editItemData.value.name.trim() === '') {
        errorMessage.value = "Saisir un produit";
        return;
      }
      if (editItemData.value.quantity > 10) {
        errorMessage.value = 'Quantité max : 10';
        return;
      }
      if (editItemData.value.name.trim() !== '' && editItemData.value.quantity > 0 && editItemData.value.quantity <= 10 && editItemData.value.name.length <= 30) {
        items.value[currentEditIndex.value] = { ...editItemData.value };
        saveItems();
        closeEditModal();
        errorMessage.value = '';
      }
    };

    const filterInput = () => {
      newItem.value.name = newItem.value.name.replace(/[0-9]/g, '');
    };

    const filterEditInput = () => {
      editItemData.value.name = editItemData.value.name.replace(/[0-9]/g, '');
    };

    const clearAll = () => {
      items.value = [];
      saveItems();
    };

    const changeFont = () => {
      const fontMap = {
        "'Bastian Script', cursive": 'bastian-script',
        "'OpenDyslexic', sans-serif": 'open-dyslexic',
      };
      const fontName = fontMap[selectedFont.value];
      const fontUrl = `https://www.cdnfonts.com/${fontName}.font`;
      const link = document.createElement('link');
      link.href = fontUrl;
      link.rel = 'stylesheet';
      document.head.appendChild(link);
    };

    onMounted(changeFont);

    watch(items, saveItems, { deep: true });

    return {
      newItem,
      items,
      addItem,
      removeItem,
      togglePurchased,
      editItemData,
      openEditModal,
      closeEditModal,
      saveEdit,
      clearAll,
      selectedFont,
      changeFont,
      filterInput,
      filterEditInput,
      showEditModal,
      errorMessage
    };
  }
};
</script>

<style>
@import url('https://www.cdnfonts.com/bastian-script.font');
@import url('https://www.cdnfonts.com/open-dyslexic.font');

* {
  scrollbar-width: thin;
}

body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  overflow-x: hidden;
}

.shopping-list-background {
  position: relative;
  width: 100vw;
  min-height: 100vh;
  background: linear-gradient(to bottom, #1e3c72, #2a5298);
  overflow: hidden;
}

.bubbles {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.bubble {
  position: absolute;
  bottom: -100px;
  width: 40px;
  height: 40px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  opacity: 0.6;
  animation: rise 10s infinite ease-in-out;
}

.bubble:nth-child(1) {
  left: 10%;
  animation-duration: 8s;
}

.bubble:nth-child(2) {
  left: 20%;
  animation-duration: 10s;
  animation-delay: 2s;
}

.bubble:nth-child(3) {
  left: 25%;
  animation-duration: 12s;
  animation-delay: 4s;
}

.bubble:nth-child(4) {
  left: 40%;
  animation-duration: 14s;
  animation-delay: 6s;
}

.bubble:nth-child(5) {
  left: 70%;
  animation-duration: 16s;
  animation-delay: 8s;
}

.bubble:nth-child(6) {
  left: 80%;
  animation-duration: 18s;
  animation-delay: 10s;
}

@keyframes rise {
  0% {
    bottom: -100px;
    transform: translateX(0);
  }
  50% {
    transform: translateX(20px);
  }
  100% {
    bottom: 100vh;
    transform: translateX(-20px);
  }
}

.shopping-list-container {
  position: relative;
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  z-index: 1;
}


.title {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

.input-group {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.input-group input[type="text"] {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px 0 0 4px;
  outline: none;
  flex: 1;
}

.quantity-input {
  width: 60px;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-left: none;
  border-right: none;
  outline: none;
  text-align: center;
}

.input-group button {
  padding: 10px 20px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-left: none;
  background: #28a745;
  color: white;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.input-group button:hover {
  background: #218838;
}

.font-selector {
  text-align: center;
  margin-bottom: 20px;
}

.font-select {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  outline: none;
  background: #fff;
  margin-left: 10px;
  cursor: pointer;
}

.shopping-list {
  list-style: none;
  padding: 0;
}

.shopping-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
  background: #fff;
  transition: background 0.3s;
}

.shopping-list li:hover {
  background: #f1f1f1;
}

.purchased {
  text-decoration: line-through;
  color: #888;
}

.icons {
  display: flex;
  gap: 10px;
}

.icon {
  cursor: pointer;
}

.edit-icon {
  color: #ffc107;
}

.delete-icon {
  color: #dc3545;
}

.clear-button {
  display: block;
  margin: 20px auto 0;
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  background: #dc3545;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

.clear-button:hover {
  background: #c82333;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;
}

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-content h2 {
  margin-top: 0;
}

.modal-content label {
  display: block;
  margin: 10px 0 5px;
}

.modal-content input {
  width: 90%;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.modal-content button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  margin-right: 10px;
  cursor: pointer;
}

.modal-content button:first-of-type {
  background: #28a745;
  color: white;
}

.modal-content button:last-of-type {
  background: #dc3545;
  color: white;
}

@media (max-width: 700px) {
  .shopping-list-container{
    margin: 50px 15px;
  }
}


@media (max-width: 600px) {
  .modal-content {
    width: 90%;
    padding: 20px;
    max-width: 400px;
  }

  .modal-content input {
    padding: 8px;
  }

  .modal-content button {
    padding: 8px 16px;
  }

}

@media (max-width: 500px) {
  .modal-content {
    width: 80%;
    padding: 20px;
    max-width: 400px;
  }

}
.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
</style>
