<!-- The custom element's tag name. -->
<svelte:options tag="svelte-to-do-list-preview" />

<script>
  // Import required Svelte module.
  import {onMount} from 'svelte';
  // Export the ID attribute value to make it accessible.
  export let id = undefined;
  // Export the to-do list property to make it accessible.
  export let todolist = undefined;
  // Boolean to hold off rendering until the required data is available.
  let readyToRender = false;
  let countToDoListItemsAll;
  let countToDoListItemsPending;
  let countToDoListItemsDone;
  onMount(() => {
    const toDoListId = +id;
    todolist = {};
    // Retrieve a to-do lists array from local storage if available or create an empty one.
    const toDoLists = JSON.parse(localStorage.getItem('toDoLists')) || [];
    if (toDoLists.length > 0) {
      // Within the to-do lists array find the to-do list associated with the passed ID.
      todolist = toDoLists.find(toDoList => {
        return toDoList.id === toDoListId;
      }) || {};
    }
    // In case there are no to-do list items assign an empty array.
    todolist.items = todolist.items || [];
    if (todolist.items.length > 0) {
      const toDoListItemsPending = todolist.items.filter(toDoListItem => {
        return toDoListItem.done !== true;
      });
      const toDoListItemsDone = todolist.items.filter(toDoListItem => {
        return toDoListItem.done !== false;
      });
      countToDoListItemsAll = todolist.items.length;
      countToDoListItemsPending = toDoListItemsPending.length;
      countToDoListItemsDone = toDoListItemsDone.length;
    }
    readyToRender = true;
  });
</script>

{#if readyToRender}
<section class="container">
  <header>
    <h1>t<span id="first-o">o</span>-d<span id="second-o">o</span> list</h1>
    {#if todolist.name !== undefined && todolist.done !== undefined}<h2 class="{!todolist.done ? 'pending' : 'done'}">{todolist.name}</h2>{/if}
  </header>
  <section class="to-do-list">
    {#if countToDoListItemsAll > 0}
    <ul class="to-do-list">
      {#each todolist.items as toDoListItem}
      <li id="{toDoListItem.id}" class="{toDoListItem.done ? 'done' : 'pending'}">
        <input class="select" type="checkbox" bind:checked={toDoListItem.done}>
        <label data-created="{toDoListItem.created}" data-last-modified="{toDoListItem.lastUpdated}" class="text">{toDoListItem.text}</label>
      </li>
      {/each}
    </ul>
    {:else if todolist.id !== undefined}
    <p>Good for you: There's nothing to do!</p>
    {:else}
    <p>Invalid ID!</p>
    {/if}
  </section>
  <hr class="{countToDoListItemsAll > 0 ? '' : 'hidden'}">
  <footer class="{countToDoListItemsAll > 0 ? '' : 'hidden'}">
    <span id="count" class="{countToDoListItemsAll > 0 ? '' : 'hide hidden'}">{countToDoListItemsAll} ({countToDoListItemsPending}/{countToDoListItemsDone})</span>
  </footer>
</section>
{/if}

<style>
  :host {
    --cyan: rgba(0, 157, 224, 1.0);
    --gray: rgba(128, 128, 128, 1.0);
    font-family: 'Montserrat', sans-serif;
    text-align: center;
    color: initial;
  }
  .hidden {
    display: none;
  }
  header h1 {
    position: relative;
    margin: 0.25em 0 0 0;
    font-size: 3em;
    font-family: 'Arvo', sans-serif;
    letter-spacing: 0.05em;
  }
  header h1 span#first-o,
  header h1 span#second-o {
    display: inline-block;
    color: transparent;
  }
  header h1 span#first-o::before,
  header h1 span#second-o::before {
    content: '';
    display: inline-block;
    margin: 0 -0.6em 0 0;
    border-radius: 50%;
    box-sizing: border-box;
    width: 0.55em;
    height: 0.55em;
  }
  header h1 span#first-o::before {
    border: 0.12em solid var(--cyan);
    background: transparent;
  }
  header h1 span#second-o::before {
    background: var(--cyan);
  }
  header h1::after {
    content: '';
    position: absolute;
    margin: 0 0 0 0.25em;
    width: 0.5em;
    height: 0.5em;
    background: url('/img/svelte.svg') no-repeat;
  }
  header h2 {
    margin: 0;
  }
  header h2.done {
    display: inline-block;
    background: linear-gradient(to right, transparent 0, currentcolor 0) no-repeat right center / 100% 0.1em;
    color: var(--gray);
  }
  section.to-do-list ul.to-do-list {
    padding: 0;
    text-align: start;
    list-style: none;
  }
  section.to-do-list ul.to-do-list li {
    position: relative;
    padding: 0.5em;
  }
  section.to-do-list ul.to-do-list li input[type=checkbox].select {
    appearance: none;
    position: relative;
    bottom: 0.1em;
    margin: 0 0.5em 0 0;
    font-size: inherit;
    vertical-align: middle;
  }
  section.to-do-list ul.to-do-list li.pending input[type=checkbox].select,
  section.to-do-list ul.to-do-list li.done input[type=checkbox].select {
    display: inline-block;
    border-radius: 50%;
    box-sizing: border-box;
    width: 1em;
    height: 1em;
  }
  section.to-do-list ul.to-do-list li.pending input[type=checkbox].select {
    border: 0.2em solid var(--cyan);
    background: transparent;
  }
  section.to-do-list ul.to-do-list li.done input[type=checkbox].select {
    border: 0.5em solid var(--cyan);
    background: var(--cyan);
  }
  section.to-do-list ul.to-do-list li label.text {
    --strikethrough: 0;
    background: linear-gradient(to right, transparent 0, currentcolor 0) no-repeat right center / calc(var(--strikethrough) * 100%) 0.1em;
    background-position-x: left;
    cursor: text;
    word-break: break-all;
  }
  section.to-do-list ul.to-do-list li.done label.text {
    --strikethrough: 1;
    color: var(--gray);
  }
  footer {
    margin: 1em 0 3em 0;
  }
  footer span#count {
    position: relative;
    float: left;
    opacity: 1;
    padding: 0;
    color: var(--gray);
  }
</style>