<template>
  <div class="todos-container">
    <div class="todo-list">
      <div class="manual">
        <div class="filters">
          <h3>Filters&Search</h3>
          <div class="filters-bar">
            <div class="filter-group">
              <label for="statusFilter" class="fixed-label">Status:</label>
              <select v-model="statusFilter" id="statusFilter">
                <option value="all">All</option>
                <option value="true">Completed</option>
                <option value="false">Uncompleted</option>
              </select>
            </div>

            <div class="filter-group">
              <label for="userFilter" class="fixed-label">User ID:</label>
              <select v-model="userFilter" id="userFilter">
                <option value="all">All Users</option>
                <option v-for="user in users" :key="user.id" :value="user.id">{{ user.id }}</option>
              </select>
            </div>

            <div class="filter-group favourites-check">
              <label for="favouritesFilter" class="fixed-label">Favourites:</label>
              <input type="checkbox" v-model="favouritesFilter" />
            </div>
          </div>

          <div class="search-bar">
            <input v-model="searchQuery" type="text" placeholder="Search by title" />
          </div>
        </div>

        <div class="create-todo">
          <h3>Create todo</h3>
          <div class="create-block">
            <div class="create-group" >
              <label for="userId" class="fixed-label">User ID:</label>
              <input v-model="newTodo.userId" type="number" min="1" step="1" id="userId" />
            </div>
            <div class="create-group">
              <label for="title" class="fixed-label">Title:</label>
              <input v-model="newTodo.title" type="text" id="title" />
            </div>
            <div class="create-button-block">
              <button @click="addTodo">Add</button>
            </div>
          </div>
        </div>
      </div>

      <h3>Todo list</h3>
      <div v-for="todo in filteredTodos" :key="todo.id" class="todo-item">
        <input class="todos-checked" type="checkbox" @click.prevent v-model="todo.completed" />
        <div class="todo-title">{{ todo.title }}</div>
        <button class="todo-favourite" :class="favourites.includes(todo.id.toString()) ?  'todo-favourite-checked' : '' " @click="toggleFavorite(todo.id)">
          <span v-if="favourites.includes(todo.id.toString())">Favourite</span>
          <span v-else>Add to favourite</span>
        </button>
      </div>
    </div>

    <form class="create-user-form">
      <h3>USER</h3>
      <div class="form-group">
        <label for="userName" class="fixed-label">Name:</label>
        <input v-model="user.name" type="text" id="userName" />
      </div>

      <div class="form-group">
        <label for="userUsername" class="fixed-label">Username:</label>
        <input v-model="user.username" type="text" id="userUsername" />
      </div>

      <div class="form-group">
        <label for="userEmail" class="fixed-label">Email:</label>
        <input v-model="user.email" type="email" id="userEmail" />
      </div>

      <div class="form-group">
        <label for="userStreet" class="fixed-label">Street:</label>
        <input v-model="user.address.street" type="text" id="userStreet" />
      </div>

      <div class="form-group">
        <label for="userSuite" class="fixed-label">Suite:</label>
        <input v-model="user.address.suite" type="text" id="userSuite" />
      </div>

      <div class="form-group">
        <label for="userCity" class="fixed-label">City:</label>
        <input v-model="user.address.city" type="text" id="userCity" />
      </div>

      <div class="form-group">
        <label for="userZipcode" class="fixed-label">Zipcode:</label>
        <input v-model="user.address.zipcode" type="text" id="userZipcode" />
      </div>

      <div class="form-group">
        <label for="userLat" class="fixed-label">Latitude:</label>
        <input v-model="user.address.geo.lat" type="text" id="userLat" />
      </div>

      <div class="form-group">
        <label for="userLng" class="fixed-label">Longitude:</label>
        <input v-model="user.address.geo.lng" type="text" id="userLng" />
      </div>

      <div class="form-group">
        <label for="userPhone" class="fixed-label">Phone:</label>
        <input v-model="user.phone" type="text" id="userPhone" />
      </div>

      <div class="form-group">
        <label for="userWebsite" class="fixed-label">Website:</label>
        <input v-model="user.website" type="text" id="userWebsite" />
      </div>
    </form>
  </div>
</template>

<script>
import { ref, reactive, onMounted, computed } from 'vue';

export default {
  setup() {
    const todos = ref([]);
    const users = ref([]);
    const statusFilter = ref('all');
    const userFilter = ref('all');
    const searchQuery = ref('');
    const favouritesFilter = ref(false);
    const newTodo = reactive({
      userId: null,
      title: '',
      completed: false,
      favorite: false,
    });

    const filteredTodos = computed(() => {
      return todos.value.filter((todo) => {
        const userIdFilter =
            userFilter.value === 'all' || todo.userId === Number(userFilter.value);
        const statusFilterMatch =
            statusFilter.value === 'all' ||
            todo.completed.toString() === statusFilter.value;
        const titleFilter = todo.title.includes(searchQuery.value);
        const favouritesFilterMatch =
            favouritesFilter.value ? favourites.value.includes(todo.id.toString()) : true;

        return userIdFilter && statusFilterMatch && titleFilter && favouritesFilterMatch;
      });
    });

    const favourites = ref(localStorage.getItem('favourites')?.split(',') || []);
    const user = JSON.parse(localStorage.getItem('user')) || {};

    const fetchTodos = async () => {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/todos');
        todos.value = await response.json();
      } catch (error) {
        console.error(error);
      }
    };

    onMounted(() => {
      fetchTodos();
      fetchUsers();
    });

    const fetchUsers = async () => {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        users.value = await response.json();
      } catch (error) {
        console.error(error);
      }
    };

    const addTodo = async () => {
      try {
        const url = 'https://jsonplaceholder.typicode.com/todos';
        const data = {
          id: newTodo.userId,
          title: newTodo.title,
        };

        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(data),
        });

        const responseData = await response.json();
        responseData.userId = newTodo.userId;
        responseData.completed = false;
        todos.value.push(responseData);
        window.scrollTo({
          top: document.body.scrollHeight,
          behavior: "smooth",
        });
        localStorage.setItem('newToDoId', responseData.id);
        newTodo.userId = null;
        newTodo.title = ''
      } catch (error) {
        console.error(error);
      }
    };

    const toggleFavorite = (id) => {
      if (!favourites.value.includes(id.toString())) {
        favourites.value.push(id.toString());
      } else {
        favourites.value = favourites.value.filter((favId) => favId !== id.toString());
      }
      localStorage.setItem('favourites', favourites.value);
    };

    return {
      todos,
      users,
      statusFilter,
      userFilter,
      searchQuery,
      newTodo,
      user,
      fetchTodos,
      fetchUsers,
      addTodo,
      toggleFavorite,
      filteredTodos,
      favourites,
      favouritesFilter,
    };
  },
};
</script>

<style lang="scss" scoped>
.todos-container {
  border-radius: 5px;
  background-color: #f0f0f0;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  margin-bottom: 40px;
  @media screen and (max-width: 968px) {
    flex-direction: column;
  }
}
.todos-checked {
  flex: 1
}
.todo-title {
  flex: 10;
  text-align: left;
}

.todo-favourite {
  width: 120px;

  &-checked {
    background-color: brown;
  }
}

.form-group {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.filter-group, .create-group {
  @media screen and (max-width: 968px) {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
  }
}

.create-user-form {
  margin-right: 20px;

}

.create-button-block {
  @media screen and (max-width: 968px) {
    align-self: flex-end;
  }
}

.create-todo,
.todo-list {
  @media screen and (max-width: 968px) {
    order:2
  }
}

.fixed-label {
  width: 80px;
  text-align: left;
}

label {
  margin-right: 10px;
}

select,
input {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 3px;
}

.todo-list {
  display: flex;
  flex-direction: column;
}

.todo-item {
  display: flex;
  align-items: flex-start;
  margin-bottom: 10px;
}

button {
  padding: 10px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

.create-block,
.filters-bar {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  @media screen and (max-width: 968px) {
    flex-direction: column;
    align-items: flex-start;
    div {
      margin-top: 10px;
    }
  }
}

.create-block {
  button {
    width: 120px;
  }
}

.search-bar {
  margin-top: 15px;
  input {
    width: calc(100% - 16px);
    @media screen and (max-width: 968px) {
      width: 95%;
    }
  }


}

.favourites-check {
  display: flex;
  align-items: center;
}
</style>
