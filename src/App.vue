<template>
  <div>
    <h1>Masukan Kata</h1>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>
    <ul>
      <li v-for="article in articles" :key="article.id">
        <strong>{{ article.title }}</strong
        ><br />
        {{ article.content }}<br />
        <button @click="edit(article)">Edit</button><br />
        <button @click="remove(article.id)" style="background-color: red">
          Hapus
        </button>
      </li>
    </ul>
    <button @click="load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

export default {
  setup() {
    const form = reactive({
      id: null,
      title: "",
      content: "",
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get("http://localhost:3000/articles");
        articles.value = response.data;
      } catch (error) {
        console.error("Error loading articles: ", error);
      }
    }

    async function save() {
      try {
        let url, method;
        if (form.id === null) {
          url = "http://localhost:3000/articles";
          method = "post";
        } else {
          url = "http://localhost:3000/articles/" + form.id;
          method = "put";
        }
        const response = await axios[method](url, form);

        if (form.id === null) {
          // New article
          articles.value.push(response.data);
        } else {
          // Update existing article
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        }
        // Reset form
        form.id = null;
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error("Error saving article: ", error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (error) {
        console.error("Error deleting article: ", error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, remove, edit, load };
  },
};
</script>

<style scoped>
div {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

form {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

input[type="text"],
textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #0056b3;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  background: #f9f9f9;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

li:hover {
  background-color: #f1f1f1;
}

li button {
  margin-top: 10px;
  background-color: #28a745;
}

li button:hover {
  background-color: #218838;
}
</style>