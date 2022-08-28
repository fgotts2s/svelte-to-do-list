<!-- The custom element's tag name. -->
<svelte:options tag="svelte-to-do-list" />

<script>
  // Import required Svelte modules.
  import {onMount, onDestroy} from 'svelte';
  // Export the ID attribute value to make it accessible.
  export let id;
  const toDoListId = +id;
  let toDoList = {};
  // Retrieve a to-do lists array from local storage if available or create an empty one.
  let toDoLists = JSON.parse(localStorage.getItem('toDoLists')) || [];
  if (toDoLists.length > 0) {
    // Within the to-do lists array find the to-do list associated with the passed ID.
    toDoList = toDoLists.find(toDoList => {
      return toDoList.id === toDoListId;
    }) || {};
  }
  // In case there are no to-do list items assign an empty array.
  toDoList.items = toDoList.items || [];
  // Allow for making the create to-do list input field accessible.
  let input;
  // Prepare the window location hash for being reactive.
  $: windowLocationHash = undefined;
  // Make the count of all to-do list items reactive.
  $: countToDoListItemsAll = toDoList.items.length;
  // Make the count of pending and done to-do list items reactive.
  let countToDoListItemsPending;
  let countToDoListItemsDone;
  $: if (toDoList.items.length > 0) {
    const toDoListItemsPending = toDoList.items.filter(toDoListItem => {
      return toDoListItem.done !== true;
    });
    const toDoListItemsDone = toDoList.items.filter(toDoListItem => {
      return toDoListItem.done !== false;
    });
    countToDoListItemsPending = toDoListItemsPending.length;
    countToDoListItemsDone = toDoListItemsDone.length;
  }
  onMount(() => {
    // Determine the window location hash.
    windowLocationHash = window.location.hash === '' ? '#all' : window.location.hash;
    // Add an event handler for when the window hash changed.
    window.addEventListener('hashchange', handleHashChange);
    input.focus();
  });
  onDestroy(() => {
    // Remove the event handler for when the window hash changed.
    window.removeEventListener('hashchange', handleHashChange);
  });
  /**
   * Method to store changed to-do list to local storage.
   * @param {object} toDoListChanged - The changed to-do list to store.
   */
  function store(toDoListChanged) {
    toDoLists = toDoLists.map(toDoList => {
      if (toDoList.id === toDoListId) {
        const toDoListDateLastUpdated = new Date();
        toDoList.lastUpdated = toDoListDateLastUpdated.toJSON();
        toDoList.items = toDoListChanged.items;
      }
      return toDoList;
    });
    localStorage.setItem('toDoLists', JSON.stringify(toDoLists));
  }
  /**
   * Method to create a new to-do list item.
   * @param {object} event - The event that triggered this method.
   */
  function createToDoListItem(event) {
    if (event.key === 'Enter') {
      if (!event.target.value.match(/^\s*$/)) {
        const toDoListItemDateCreated = new Date();
        const toDoListItem = {
          id: toDoListItemDateCreated.getTime(),
          created: toDoListItemDateCreated.toJSON(),
          lastUpdated: null,
          text: event.target.value,
          done: false
        };
        toDoList.items = [...toDoList.items, toDoListItem];
        store(toDoList);
        event.target.value = '';
        event.target.focus();
      }
    }
  }
  /**
   * Method to handle the update of a to-do list item.
   * @param {object} nodeToDoListItemText - The current to-do list item's text node.
   * @param {object} nodeInputUpdateToDoListItem - The new to-do list item's text node.
   */
  function handleToDoListItemUpdate(nodeToDoListItemText, nodeInputUpdateToDoListItem) {
    // Only accept input that isn't all whitespace.
    if (!nodeInputUpdateToDoListItem.value.match(/^\s*$/)) {
      if (nodeInputUpdateToDoListItem.value !== nodeToDoListItemText.textContent) {
        toDoList.items = toDoList.items.map(toDoListItem => {
          if (toDoListItem.id === +nodeInputUpdateToDoListItem.parentNode.id) {
            const toDoListItemDateLastUpdated = new Date();
            toDoListItem.lastUpdated = toDoListItemDateLastUpdated.toJSON();
            toDoListItem.text = nodeInputUpdateToDoListItem.value;
          }
          return toDoListItem;
        });
      }
      nodeInputUpdateToDoListItem.remove();
      nodeToDoListItemText.classList.remove('hidden');
      store(toDoList);
      input.focus();
    }
  }
  /**
   * Method to update a to-do list item.
   * @param {object} event - The event that triggered this method.
   */
  function updateToDoListItem(event) {
    const nodeToDoListItemText = event.target;
    const nodeInputUpdateToDoListItem = document.createElement('input');
	  nodeInputUpdateToDoListItem.style = 'width: calc(100% - 1.5em)';
    nodeInputUpdateToDoListItem.type = 'text';
    nodeInputUpdateToDoListItem.value = nodeToDoListItemText.textContent;
    nodeToDoListItemText.parentNode.appendChild(nodeInputUpdateToDoListItem);
    nodeToDoListItemText.classList.add('hidden');
    nodeInputUpdateToDoListItem.focus();
    nodeInputUpdateToDoListItem.addEventListener('blur', event => {
      handleToDoListItemUpdate(nodeToDoListItemText, nodeInputUpdateToDoListItem);
    });
    nodeInputUpdateToDoListItem.addEventListener('keydown', event => {
      if (event.key === 'Enter') {
        handleToDoListItemUpdate(nodeToDoListItemText, nodeInputUpdateToDoListItem);
      } else if (event.key === 'Escape') {
        nodeInputUpdateToDoListItem.remove();
        nodeToDoListItemText.classList.remove('hidden');
      }
    });
  }
  /**
   * Method to delete a to-do list item.
   * @param {number} id - The to-do list item's ID.
   */
  function deleteToDoListItem(id) {
    toDoList.items = toDoList.items.filter(toDoListItem => {
      return toDoListItem.id !== id;
    });
    store(toDoList);
  }
  /**
   * Method to toggle a to-do list item.
   * @param {number} id - The to-do list item's ID.
   */
  function toggleToDoListItem(id) {
    toDoList.items = toDoList.items.map(toDoListItem => {
      if (toDoListItem.id === id) {
        toDoListItem.done = !toDoListItem.done;
      }
      return toDoListItem;
    });
    store(toDoList);
  }
  /**
   * Method to toggle all to-do list items.
   */
  function toggleAllToDoListItems() {
    toDoList.items = toDoList.items.map(toDoListItem => {
      toDoListItem.done = !toDoListItem.done;
      return toDoListItem;
    });
    store(toDoList);
  }
  /**
   * Method to clear (delete all done) to-do list (items).
   * @param {object} event - The event that triggered this method.
   */
  function clearToDoList(event) {
    event.target.classList.add('hidden');
    event.target.nextElementSibling.classList.remove('hidden');
  }
  /**
   * Method to confirm the clearance (the deletion of all done) of the to-do list (items).
   * @param {object} event - The event that triggered this method.
   */
  function clearToDoListYes(event) {
    event.target.parentNode.classList.add('hidden');
    toDoList.items = toDoList.items.filter(toDoListItem => {
      return toDoListItem.done === false;
    });
    store(toDoList);
    input.focus();
  }
  /**
   * Method to reject the clearance (the deletion of all done) of the to-do list (items).
   * @param {object} event - The event that triggered this method.
   */
  function clearToDoListNo(event) {
    event.target.parentNode.classList.add('hidden');
    event.target.parentNode.previousElementSibling.classList.remove('hidden');
  }
  /**
   * Method to handle the window hash change event.
   */
  const handleHashChange = () => {
    windowLocationHash = window.location.hash;
  }
</script>

<section class="container">
  <header>
    <h1>t<span id="first-o">o</span>-d<span id="second-o">o</span> list</h1>
    {#if toDoList.name !== undefined && toDoList.done !== undefined}<h2 class="{!toDoList.done ? 'pending' : 'done'}">{toDoList.name}</h2>{/if}
    <button on:click={toggleAllToDoListItems} id="toggle-all" class="{countToDoListItemsAll > 0 ? '' : 'hide'}" title="Click to toggle all"></button>
    <input on:keydown={createToDoListItem} bind:this={input} id="create" class="{countToDoListItemsAll > 0 ? 'shrink' : ''}" type="text" placeholder="What do you have to do?">
  </header>
  <section class="to-do-list">
    {#if countToDoListItemsAll > 0}
    <ul class="to-do-list">
      {#each toDoList.items as toDoListItem}
      {#if (windowLocationHash === '#all') || (windowLocationHash === '#pending' && !toDoListItem.done) || (windowLocationHash === '#done' && toDoListItem.done)}
      <li id="{toDoListItem.id}" class="{toDoListItem.done ? 'done' : 'pending'}">
        <input on:click={toggleToDoListItem(toDoListItem.id)} class="select" title="Click to mark as {toDoListItem.done === false ? 'done' : 'pending'}" type="checkbox" bind:checked={toDoListItem.done}><!--
     --><label on:click={updateToDoListItem} data-created="{toDoListItem.created}" data-last-modified="{toDoListItem.lastUpdated}" class="text" title="Click to edit">{toDoListItem.text}</label><!--
     --><button on:click={deleteToDoListItem(toDoListItem.id)} class="delete" title="Click to delete"></button><!--
   --></li>
      {/if}
      {/each}
    </ul>
    {:else if toDoList.id !== undefined}
    <p>Good for you: There's nothing to do!</p>
    {:else}
    <p>Invalid ID! Your entries won't be stored!</p>
    {/if}
  </section>
  <hr class="{countToDoListItemsAll > 0 ? '' : 'hidden'}">
  <footer class="{countToDoListItemsAll > 0 ? '' : 'hidden'}">
    <span id="count" class="{countToDoListItemsAll > 0 ? '' : 'hide hidden'}">{countToDoListItemsAll} ({countToDoListItemsPending}/{countToDoListItemsDone})</span>
    <ul class="filters">
      <li><a id="all" class="{windowLocationHash === '#all' ? 'selected' : ''}" href="#all">All</a></li>
      <li><a id="pending" class="{windowLocationHash === '#pending' ? 'selected' : ''}" href="#pending">Pending</a></li>
      <li><a id="done" class="{windowLocationHash === '#done' ? 'selected' : ''}" href="#done">Done</a></li>
    </ul>
    <button on:click={clearToDoList} id="clear" class="{countToDoListItemsDone > 0 ? '' : 'hide hidden'}" title="Click to delete done">Clear</button>
    <span id="confirmation" class="hidden">
      Sure?
      <button on:click={clearToDoListYes} id="yes" title="Click to confirm">Yes</button>
      <button on:click={clearToDoListNo} id="no" title="Click to reject">No</button>
    </span>
  </footer>
</section>

<style>
  :host {
    --cyan: rgba(0, 157, 224, 1.0);
    --cyan-three-quarter-transparent: rgba(0, 157, 224, 0.25);
    --gray: rgba(128, 128, 128, 1.0);
    --gray-semi-transparent: rgba(128, 128, 128, 0.5);
    --short: 250ms;
    --long: 500ms;
    font-family: 'Montserrat', sans-serif;
    text-align: center;
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
    background: url('/svelte-to-do-list/img/svelte.svg') no-repeat;
  }
  header h2 {
    margin: 0;
  }
  header h2.done {
    display: inline-block;
    background: linear-gradient(to right, transparent 0, currentcolor 0) no-repeat right center / 100% 0.1em;
    color: var(--gray);
  }
  header button#toggle-all {
    all: unset;
    position: relative;
    top: 1.75em;
    justify-content: space-around;
    display: flex;
    opacity: 1;
    transition: opacity var(--short) ease-in-out var(--short), transform var(--short) ease-in-out var(--short);
    margin: 0 0 0 0.5em;
    width: 1em;
    height: 1em;
    cursor: pointer;
  }
  header button#toggle-all::before,
  header button#toggle-all::after {
    content: '';
    position: absolute;
    border-radius: 0.1em;
    width: 0.2em;
    height: 1em;
    background: var(--cyan);
  }
  header button#toggle-all::before {
    transform: translate(20%) rotate(-45deg);
    transform-origin: bottom right;
  }
  header button#toggle-all::after {
    transform: translate(-20%) rotate(45deg);
    transform-origin: bottom left;
  }
  header button#toggle-all.hide {
    opacity: 0;
    transform: rotate(-90deg);
  }
  input[type=text] {
    position: relative;
    transition: margin var(--short) ease-in-out var(--short), width var(--short) ease-in-out var(--short);
    border: 0.15em solid var(--cyan-three-quarter-transparent);
    border-radius: 0.5em;
    box-sizing: border-box;
    width: 100%;
    padding: 0.5em;
    font-size: 1em;
    font-family: inherit;
    outline: none;
  }
  input[type=text]:focus {
    border-color: var(--cyan);
  }
  header input[type=text].shrink {
    margin: 0 0 0 10%;
    width: 90%;
  }
  section.to-do-list ul.to-do-list {
    padding: 0;
    text-align: start;
    list-style: none;
  }
  section.to-do-list ul.to-do-list li {
    position: relative;
    transition: opacity var(--long) ease-in-out, outline-color var(--short) ease-in-out;
    padding: 0.5em;
    outline: transparent dashed 0.1em;
    border-radius: 0.5em;
  }
  section.to-do-list ul.to-do-list li:hover {
    outline: var(--gray-semi-transparent) dashed 0.1em;
  }
  section.to-do-list ul.to-do-list li input[type=checkbox].select {
    appearance: none;
    position: relative;
    bottom: 0.1em;
    transition: background var(--long) ease-in-out, border var(--long) ease-in-out;
    margin: 0 0.5em 0 0;
    cursor: pointer;
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
    transition: background-size var(--long) ease-in-out, color var(--long) ease-in-out;
    background: linear-gradient(to right, transparent 0, currentcolor 0) no-repeat right center / calc(var(--strikethrough) * 100%) 0.1em;
    background-position-x: left;
    cursor: text;
    word-break: break-all;
  }
  section.to-do-list ul.to-do-list li.done label.text {
    --strikethrough: 1;
    color: var(--gray);
  }
  section.to-do-list ul.to-do-list li button.delete {
    all: unset;
    position: absolute;
    bottom: 0.6em;
    right: 0;
    justify-content: space-around;
    display: flex;
    opacity: 0;
    transition: opacity var(--short) ease-in-out;
    margin: 0 0.5em 0 0;
    width: 1em;
    height: 1em;
    cursor: pointer;
  }
  section.to-do-list ul.to-do-list li button.delete::before,
  section.to-do-list ul.to-do-list li button.delete::after {
    content: '';
    position: absolute;
    border-radius: 0.1em;
    width: 0.2em;
    height: 1em;
    background: var(--cyan);
  }
  section.to-do-list ul.to-do-list li button.delete::before {
    transform: rotate(45deg);
  }
  section.to-do-list ul.to-do-list li button.delete::after {
    transform: rotate(-45deg);
  }
  section.to-do-list ul.to-do-list li:hover button.delete {
    opacity: 1;
  }
  :global(section.to-do-list ul.to-do-list li input[type=text]) {
    width: calc(100% - 1.5em);
  }
  footer {
    margin: 1em 0 3em 0;
  }
  footer span#count {
    position: relative;
    float: left;
    opacity: 1;
    transition: opacity var(--long) ease-in-out;
    padding: 0;
    color: var(--gray);
  }
  footer span#count.hide {
    opacity: 0;
  }
  footer ul.filters {
    position: absolute;
    left: 0;
    right: 0;
    margin: 0;
    padding: 0;
    list-style: none;
  }
  footer ul.filters li {
    display: inline;
  }
  footer ul.filters li a:hover {
    outline: var(--gray-semi-transparent) solid 0.1em;
    border-radius: 0.25em;
  }
  footer a {
    padding: 0.1em 0.2em;
    cursor: pointer;
    text-decoration: none;
    color: var(--cyan);
  }
  footer a.selected {
    outline: var(--gray) solid 0.1em;
    border-radius: 0.25em;
  }
  footer button#clear,
  footer button#yes,
  footer button#no {
    all: unset;
    cursor: pointer;
    color: var(--cyan);
  }
  footer button#clear,
  footer span#confirmation {
    position: relative;
    float: right;
    opacity: 1;
    transition: opacity var(--long) ease-in-out;
    padding: 0;
  }
  footer button#clear:hover,
  footer button#yes:hover,
  footer button#no:hover {
    text-decoration: underline;
    text-decoration-color: var(--gray-semi-transparent);
  }
  footer button#clear.hide {
    opacity: 0;
  }
  footer button#clear.hidden {
    display: none;
  }
</style>