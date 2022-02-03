<script>
	import { initializeApp } from "firebase/app";
	import {
		getFirestore,
		collection,
		onSnapshot,
		doc,
		addDoc,
		updateDoc,
		deleteDoc,
		orderBy,
		query,
	} from "firebase/firestore";

	// import { firebaseConfig } from "$lib/firebaseConfig";
	import { onDestroy } from "svelte";

	// replace the following config with your own config on firebase console
	export const firebaseConfig = {
  apiKey: "AIzaSyDyLcYGrajkwEe7Oo6VjpLw1QSY3uC19_M",
  authDomain: "sk-fs-6c641.firebaseapp.com",
  projectId: "sk-fs-6c641",
  storageBucket: "sk-fs-6c641.appspot.com",
  messagingSenderId: "811983126544",
  appId: "1:811983126544:web:f900d70238d5a77ac18a95"
};

	const firebaseApp = initializeApp(firebaseConfig);
	const db = getFirestore();

	const todosCol = collection(db, "todos");

	let todos = [];

	const unsubscribe = onSnapshot(
		query(todosCol, orderBy("createdAt")),
		(querySnapshot) => {
			todos = querySnapshot.docs.map((doc) => {
				return { ...doc.data(), id: doc.id };
			});
		}
	);

	onDestroy(unsubscribe);

	let todoInput = "";

	async function addTodo() {
		if (!todoInput) return;

		const todo = {
			task: todoInput,
			isComplete: false,
			createdAt: new Date(),
		};

		await addDoc(collection(db, "todos"), todo);
		todoInput = "";
	}

	async function toggleTodoIsComplete(todo) {
		// const targetTodo = todos.find((todo) => todo.id === id);

		// if (!targetTodo) return;

		// targetTodo.isComplete = !targetTodo.isComplete;
		// todos = todos.map((todo) => (todo.id !== id ? todo : targetTodo));
		const docRef = doc(db, "todos", todo.id);

		await updateDoc(docRef, {
			isComplete: !todo.isComplete,
		});
	}

	async function deleteTodo(todo) {
		// todos = todos.filter((todo) => todo.id !== id);
		await deleteDoc(doc(db, "todos", todo.id));
	}

	// function generateId() {
	// 	const maxId = todos
	// 		.map((todo) => todo.id)
	// 		.reduce((acc, id) => {
	// 			return id > acc ? id : acc;
	// 		}, 0);

	// 	return maxId + 1;
	// }
</script>

<input type="text" bind:value={todoInput} placeholder="입력하세요" />
<button on:click={addTodo}>추가</button>

<ol>
	{#if todos.length > 0}
		{#each todos as todo}
			<li class:complete={todo.isComplete}>
				<span>{todo.task}</span>
				<span>
					<button on:click={() => toggleTodoIsComplete(todo)}>✓</button>
					<button on:click={() => deleteTodo(todo)}>✘</button>
				</span>
			</li>
		{/each}
	{:else}
		<p>No todos yet</p>
	{/if}
</ol>

<style>
	.complete {
		text-decoration: line-through;
	}
</style>
