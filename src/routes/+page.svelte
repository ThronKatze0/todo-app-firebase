<script lang="ts">
  import { Doc } from "sveltefire";
  import { ProgressRadial } from "@skeletonlabs/skeleton";
  import {
    Modal,
    Toast,
    getModalStore,
    initializeStores,
  } from "@skeletonlabs/skeleton";
  import { Collection } from "sveltefire";
  import { docStore } from "sveltefire";
  import { doc, updateDoc, setDoc, deleteDoc } from "firebase/firestore";
  import {
    type ModalSettings,
    type ModalStore,
    type ToastStore,
  } from "@skeletonlabs/skeleton";
  import { initializeApp } from "firebase/app";
  import { getFirestore } from "firebase/firestore";
  const app = initializeApp(/* your firebase config */);
  const firestore = getFirestore(app);
  initializeStores();

  const modalStore = getModalStore();

  let inputTodo: string = "";

  var id = 0;
  class Todo {
    name: string;
    checked: boolean;
    isEditing: boolean;
    id: number;
    constructor(
      name: string,
      checked: boolean,
      isEditing: boolean,
      idParam: number,
    ) {
      this.name = name;
      this.checked = checked;
      this.isEditing = isEditing;
      this.id = idParam == -1 ? id++ : idParam;
    }
    toString() {
      return this.name;
    }
  }

  const converter = {
    fromFirestore: (data: any) => {
      return new Todo(data.name, data.checked, data.isEditing, data.id);
    },
    toFirestore: (todo: Todo) => {
      return {
        name: todo.name,
        checked: todo.checked,
        isEditing: todo.isEditing,
      };
    },
  };

  function toTodos(data: any) {
    let todos: Todo[] = [];
    for (let i = 0; i < data.length; i++) {
      todos.push(converter.fromFirestore(data[i]));
    }
    return todos;
  }

  function addTodo() {
    const todo = new Todo(inputTodo, false, false, -1);
    let location: string = "data/storage/todos/" + todo.id;
    setDoc(doc(firestore, location), converter.toFirestore(todo));
    inputTodo = "";
  }
</script>

<Modal />

<h1 class="h1 text-center mt-6">Realtime Todo App</h1>
<form class="mt-6 flex flex-row justify-center">
  <input
    type="text"
    class="input w-1/6"
    placeholder="Todo"
    bind:value={inputTodo}
  />
  <button on:click|preventDefault={addTodo} class="btn variant-filled ml-4"
    >Add</button
  >
</form>
<div class="grid grid-cols-3 gap-4 mx-20 mt-16">
  <Collection ref={"data/storage/todos"} let:data let:count>
    <ProgressRadial slot="loading" />
    {#each toTodos(data) as todo (todo.name)}
      <div class="card">
        <header class="card-header flex flex-row justify-between mb-5">
          {#if todo.checked}
            <h2 class="h2 line-through">{todo.name}</h2>
            <input
              class="checkbox p-3.5"
              type="checkbox"
              checked
              on:click={() => {
                updateDoc(doc(firestore, "data/storage/todos/" + todo.id), {
                  checked: !todo.checked,
                });
              }}
            />
          {:else}
            <h2 class="h2">{todo.name}</h2>
            <input
              class="checkbox p-3.5"
              type="checkbox"
              on:click={() => {
                updateDoc(doc(firestore, "data/storage/todos/" + todo.id), {
                  checked: !todo.checked,
                });
              }}
            />
          {/if}
        </header>
        <section class="p-4">
          <div class="flex flex-row justify-end">
            <button
              class="btn variant-filled mr-3"
              on:click={() => {
                deleteDoc(doc(firestore, "data/storage/todos/" + todo.id));
              }}>Delete</button
            >
            <button
              class="btn variant-filled"
              on:click={() => {
                modalStore.trigger({
                  type: "prompt",
                  title: "Edit Todo",
                  body: "You can edit your Todo below",
                  value: todo.name,
                  valueAttr: {
                    type: "text",
                    minlength: 3,
                    maxlength: 30,
                    required: true,
                  },
                  response: (r) => {
                    if (r && r != todo.name) {
                      updateDoc(
                        doc(firestore, "data/storage/todos/" + todo.id),
                        {
                          name: r,
                        },
                      );
                    }
                  },
                });
              }}>Edit</button
            >
          </div>
        </section>
      </div>
    {/each}
  </Collection>
</div>
