<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'

const api = 'https://todolist-api.hexschool.io'

const signInPage = ref(false)

const isLogin = ref(false)

const selectSignUp = () => {
  signInPage.value = false
}

const selectSignIn = () => {
  signInPage.value = true
}

const signUpData = ref({
  email: '',
  password: '',
  nickname: ''
})

const uid = ref('')

const signInData = ref({
  email: '',
  password: ''
})

const userData = ref({
  nickname: '',
  uid: ''
})

const errorMsg = ref('')

const signUp = async () => {
  try {
    const res = await axios.post(`${api}/users/sign_up`, signUpData.value)
    uid.value = res.data.uid
    signInPage.value = true
    signUpData.value = {}
    errorMsg.value = ''
  } catch (error) {
    errorMsg.value = error.response.data.message  }
}

const signIn = async () => {
  try {
    const res = await axios.post(`${api}/users/sign_in`, signInData.value)
    document.cookie = `todoToken=${res.data.token};`
    checkOut()
    errorMsg.value=''
  } catch (error) {
    errorMsg.value = error.response.data.message
  }
}

const checkOut = async () => {
  try {
    const token = document.cookie.replace(/(?:(?:^|.*;\s*)todoToken\s*\=\s*([^;]*).*$)|^.*$/, '$1')
    const res = await axios.get(`${api}/users/checkout`, {
      headers: {
        Authorization: token
      }
    })
    userData.value = res.data
    isLogin.value = true
    getTodos();
  } catch (error) {
    console.log(error)
  }
}

const signOut = () => {
  document.cookie = `todoToken=true; max-age=0;`
  isLogin.value = false
  todoList.value=[]
}

const todoList = ref([])

const getTodos = async () => {
  try {
    const token = document.cookie.replace(/(?:(?:^|.*;\s*)todoToken\s*\=\s*([^;]*).*$)|^.*$/, '$1')
    const res = await axios.get(`${api}/todos/`, {
      headers: {
        Authorization: token
      }
    })
    todoList.value = res.data.data
  } catch (error) {
    console.log(error)
  }
}

const newTodo = ref({
  content: ''
})

const addTodo = async () => {
  try {
    const token = document.cookie.replace(/(?:(?:^|.*;\s*)todoToken\s*\=\s*([^;]*).*$)|^.*$/, '$1')
    const res = await axios.post(`${api}/todos/`, newTodo.value, {
      headers: {
        Authorization: token
      }
    })
    newTodo.value = {}
    getTodos()
  } catch (error) {
    console.log(error)
  }
}

const tempTodo = ref({
  content: ''
})

const prepareEdit = (todo) => {
  tempTodo.value = { ...todo }
}

const cancelEdit = () => {
  tempTodo.value = {}
}

const editTodo = async (id) => {
  try {
    const token = document.cookie.replace(/(?:(?:^|.*;\s*)todoToken\s*\=\s*([^;]*).*$)|^.*$/, '$1')
    const res = await axios.put(`${api}/todos/${id}`, tempTodo.value, {
      headers: {
        Authorization: token
      }
    })
    tempTodo.value = {}
    getTodos()
  } catch (error) {
    console.log(error)
  }
}

const deleteTodo = async (id) => {
  try {
    const token = document.cookie.replace(/(?:(?:^|.*;\s*)todoToken\s*\=\s*([^;]*).*$)|^.*$/, '$1')
    const res = await axios.delete(`${api}/todos/${id}`, {
      headers: {
        Authorization: token
      }
    })
    getTodos()
  } catch (error) {
    console.log(error)
  }
}

const updateStatus = async (id) => {
  try {
    const token = document.cookie.replace(/(?:(?:^|.*;\s*)todoToken\s*\=\s*([^;]*).*$)|^.*$/, '$1')
    const res = await axios.patch(
      `${api}/todos/${id}/toggle`,
      {},
      {
        headers: {
          Authorization: token
        }
      }
    )
    console.log(res)
    getTodos()
  } catch (error) {
    console.log(error)
  }
}

onMounted(() => {
  checkOut();
})
</script>

<template>
  <h1 class="text-center mb-5">Vue Week02</h1>
  <div v-if="isLogin" class="col-10 mx-auto">
    <div class="text-end">
      歡迎，{{ userData.nickname }}
      <button class="btn btn-outline-primary ms-2" type="button" @click="signOut">登出</button>
    </div>
  </div>
  <div v-else class="row justify-content-center">
    <div class="col-4">
      <div class="card text-center">
        <div class="card-header">
          <ul class="nav nav-tabs card-header-tabs">
            <li class="nav-item">
              <a
                class="nav-link"
                :class="[signInPage ? '' : 'active']"
                aria-current="true"
                @click="selectSignUp"
                >註冊</a
              >
            </li>
            <li class="nav-item">
              <a class="nav-link" :class="[signInPage ? 'active' : '']" @click="selectSignIn"
                >登入</a
              >
            </li>
          </ul>
        </div>
        <div class="card-body">
          <div v-if="signInPage">
            <input
              type="email"
              placeholder="Email"
              v-model="signInData.email"
              class="form-control"
            />
            <br />
            <input
              type="password"
              placeholder="Password"
              v-model="signInData.password"
              class="form-control"
            />
            <br />
            <div v-if="errorMsg" class="text-danger mb-3">{{ errorMsg }}</div>
            <button type="button" class="btn btn-primary" @click="signIn">登入</button>
          </div>
          <div v-else>
            <input
              type="email"
              placeholder="Email"
              v-model="signUpData.email"
              class="form-control"
            />
            <br />
            <input
              type="password"
              placeholder="Password"
              v-model="signUpData.password"
              class="form-control"
            />
            <br />
            <input
              type="text"
              placeholder="Nickname"
              v-model="signUpData.nickname"
              class="form-control"
            />
            <br />
            <div v-if="errorMsg" class="text-danger mb-3">{{ errorMsg }}</div>
            <button type="button" class="btn btn-primary" @click="signUp">註冊</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div v-if="isLogin" class="col-10 mx-auto">
    <h2 class="text-center">待辦事項</h2>
    <div class="input-group my-5">
      <input
        type="text"
        class="form-control"
        v-model="newTodo.content"
        placeholder="請輸入待辦內容"
      />
      <button class="btn btn-outline-primary" type="button" @click="addTodo">增加</button>
    </div>
    <table v-if="todoList.length!=0" class="table">
      <thead>
        <tr>
          <th scope="col" class="text-center col-1">#</th>
          <th scope="col" class="text-center col-6">待辦內容</th>
          <th scope="col" class="text-center col-1">狀態</th>
          <th scope="col" class="text-center col-2">操作</th>
        </tr>
      </thead>
      <tbody v-for="todo in todoList" :key="todo.id">
        <tr v-if="todo.id === tempTodo.id">
          <td></td>
          <td><input class="form-control" type="text" v-model="tempTodo.content" /></td>
          <td class="text-center align-middle">{{ todo.status ? '完成' : '未完成' }}</td>
          <td class="text-center">
            <button type="button" @click="editTodo(todo.id)" class="btn btn-primary mx-2">
              確定
            </button>
            <button type="button" @click="cancelEdit" class="btn btn-outline-primary mx-2">
              取消
            </button>
          </td>
        </tr>
        <tr v-else>
          <td class="align-middle">
            <input
              class="form-check-input"
              type="checkbox"
              style="width: 25px; height: 25px"
              @click="updateStatus(todo.id)"
              v-model="todo.status"
            />
          </td>
          <td class="text-dark align-middle" :class="[todo.status ? 'text-opacity-25' : '']">
            {{ todo.content }}
          </td>
          <td class="text-center align-middle">{{ todo.status ? '完成' : '未完成' }}</td>
          <td class="text-center">
            <button type="button" @click="prepareEdit(todo)" class="btn btn-primary mx-2">
              編輯
            </button>
            <button type="button" @click="deleteTodo(todo.id)" class="btn btn-outline-primary mx-2">
              刪除
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <h4 v-else class="text-center">目前沒有任何待辦事情</h4>
  </div>
</template>
