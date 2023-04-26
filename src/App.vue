<script setup>
import axios from 'axios';
import {  ref } from 'vue';
import { Field, Form, ErrorMessage } from 'vee-validate';
import { toTypedSchema } from '@vee-validate/zod';
import * as zod from 'zod';
import { useAxios } from '@vueuse/integrations/useAxios'

const todos = ref([]);

const todo = ref([]);

const update = ref(false);

const instance = axios.create({
  baseURL: 'http://localhost:8000/api/',
})

const validationSchema =  toTypedSchema(
        zod.object({
           todo: zod.string().nonempty('This is required'),
         })
); 

const GetTodos = async () => {
     
        const { data } = await useAxios('/todo', instance);
        todos.value = data.value.data

}
const GetTodo = async (id) => {

     update.value = true;
     const { data } = await useAxios('/todo/'+id, instance);
     todo.value = data.value.data
}

const DeleteTodo = async(id) => {
        const { execute } =  useAxios('/todo/'+id, { method: "DELETE" }, instance , { immediate:false } );
        const result =  async () => {await execute().then((res) => { GetTodos() })}
        result();
}

function onSubmit(values,actions) {

      if(update.value)
      {
        const { execute } =  useAxios('/todo/'+todo.value.id , { method: "PUT" }, instance , { immediate:false } );
        const requestBody = { name : todo.value.name , status : 0};
        const result =  async () => {await execute({ data: requestBody }).then((res) => { GetTodos() })}
        result();
        actions.setFieldValue('todo', '');
        update.value = false;
     
      }
      else
      {
        const { execute } =  useAxios('api/todo', { method: "POST" }, instance , { immediate:false } );
        const requestBody = {name : values.todo , status : 0};
        const result =  async () => {await execute({ data: requestBody }).then((res) => { GetTodos() })}
        result();
        actions.setFieldValue('todo', '');

      }           
}
GetTodos()
</script>

<template>
  
 <h1>ADD TODO LIST</h1>
			<div id="line"></div>
      <Form :validation-schema="validationSchema" @submit="onSubmit">
			<div class="task-input">
				<Field type="text" id="add-task" name="todo" placeholder="Add Task..." v-model="todo.name"/>
				<div class="flex-button">
					<input type="submit" value="➕" name='add' id="add-btn" title="Add Task">
					<input type="button" value="❌" name='delete' id="del-btn" title="Remove All Task">
				</div>
			</div>
      <ErrorMessage name="todo" />
    </Form>
			<div class="new-tasks" v-if="todos.length > 0">
				<div class="task" v-for="todo in todos">
					         <input type="text" id="added-task" :name="todo.name" disabled="" :value="todo.name">
                    <div>
                        <input type="button" value="✔️" name="update" title="update task" class="update-task">
                        <input type="button" value="✏️" name="rename" title="rename task" class="rename-task" @click="GetTodo(todo.id)">
                        <input type="button" value="❌" name="delete" title="delete task" class="del-task"  @click="DeleteTodo(todo.id)">
                    </div>
                </div>
				
			</div>
</template>


