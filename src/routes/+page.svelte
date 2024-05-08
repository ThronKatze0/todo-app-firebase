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

  function addTodo() {
    let location: string = "data/storage/todos/" + inputTodo;
    setDoc(doc(firestore, location), {
      name: inputTodo,
      checked: false,
      isEditing: false,
    });
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
  <Collection ref={"data/storage/todos"} let:data>
    {#each data as todo (todo.name)}
      <div class="card">
        <header class="card-header">
          <h2 class="h2">{todo.name}</h2>
        </header>
        <section class="p-4">
          <button
            class="btn variant-filled"
            on:click={() => {
              deleteDoc(doc(firestore, "data/storage/todos/" + todo.name));
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
                  maxlength: 10,
                  required: true,
                },
                response: (r) => {
                  if (r && r != todo.name) {
                    updateDoc(
                      doc(firestore, "data/storage/todos/" + todo.name),
                      {
                        name: r,
                      },
                    );
                  }
                },
              });
            }}>Edit</button
          >
        </section>
      </div>
    {/each}
    <ProgressRadial slot="loading" />
  </Collection>
</div>
