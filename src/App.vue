<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <h2>Buat Artikel Baru</h2>
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button save-button">Save</button>
    </form>
    <ul class="article-list">
      <h2 class="list-title">List Artikel </h2>
      <button @click="load" class="load-button">Load Articles</button>
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div v-if="editId === article.id" class="edit-container">
          <h2>Edit Article</h2>
          <input type="text" v-model="editForm.title" class="input" /><br />
          <textarea v-model="editForm.content" class="textarea"></textarea><br />
          <div class="button-group centered-buttons">
            <button @click="updateArticle(article.id)" class="button save-button">Update</button>
            <button @click="cancelEdit" class="button cancel-button">Cancel</button>
          </div>
        </div>
        <div v-else>
          <strong>{{ article.title }}</strong><br />
          <p>{{ article.content }}</p>
          <div class="button-group centered-buttons">
            <button @click="edit(article)" class="button edit-button">Edit</button>
            <button @click="deleteArticle(article.id)" class="button delete-button">Delete</button>
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>


<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });
    const editForm = reactive({
      id: null,
      title: '',
      content: ''
    });
    const articles = ref([]);
    const editId = ref(null);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, {
          title: form.title,
          content: form.content
        });

        if (method === 'post') {
          articles.value.push(response.data);
        } else {
          const index = articles.value.findIndex(article => article.id === form.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function updateArticle(id) {
      try {
        const response = await axios.put(`http://localhost:3000/articles/${id}`, {
          title: editForm.title,
          content: editForm.content
        });

        const index = articles.value.findIndex(article => article.id === id);
        if (index !== -1) {
          articles.value[index] = response.data;
        }

        cancelEdit();
      } catch (error) {
        console.error('Error updating article:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      editId.value = article.id;
      editForm.id = article.id;
      editForm.title = article.title;
      editForm.content = article.content;
    }

    function cancelEdit() {
      editId.value = null;
      editForm.id = null;
      editForm.title = '';
      editForm.content = '';
    }

    onMounted(load);

    return { form, editForm, articles, save, edit, updateArticle, deleteArticle, load, cancelEdit, editId };
  }
}
</script>

<style scoped>
.container {
  width: 80%;
  margin: 0 auto;
  padding: 50px;
}
.form {
  margin-bottom: 20px;
  color:silver;
}
.input, .textarea {
  width: 97%;
  padding: 10px;
  margin-bottom: 10px;
}
.button {
  padding: 10px 20px;
  margin-right: 10px;
}
.save-button {
  background-color: #4CAF50;
  color: white;
}
.cancel-button {
  background-color: #f44336;
  color: white;
}
.edit-button {
  background-color: #694707;
  color: white;
  padding: 10px 20px;
}
.delete-button {
  background-color: #f44336;
  color: white;
  padding: 10px 20px;
}
.load-button {
  background-color: #1f88dee4;
  color: white;
  margin-bottom: 20px;
  padding: 5px 6px;
  font-size: 16px;
  
}
.article-list {
  list-style-type: none;
  padding: 0;
  display: flex; 
  flex-wrap: wrap; 
  gap: 20px; 
}

.list-title {
  text-align: center; 
  font-size: 1.5em;
  margin-bottom: 20px; 
  color:silver;
}

.article-item {
  margin-bottom: 20px;
  padding: 10px;
  border: 1px solid #ddd;
  width: 100%; 
  height: auto; 
  display: flex; 
  flex-direction: column;
  justify-content: space-between; 
  background-color:rgba(51, 45, 45, 0.64);
}

.edit-container {
  width: 100%; 
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.button-group {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.centered-buttons {
  justify-content: center;
}
</style>
