<!-- The custom element's tag name. -->
<svelte:options tag="svelte-to-do-lists" />

<script>
  // Import required Svelte modules.
  import {onMount, onDestroy} from 'svelte';
  // Import subcomponent for a to-do list preview.
  import SvelteToDoListPreview from './SvelteToDoListPreview.svelte';
  // Retrieve a to-do lists array from local storage if available or create an empty one.
  let toDoLists = JSON.parse(localStorage.getItem('toDoLists')) || [];
  // Allow for making the whole component's DOM accessible.
  let container;
  // Allow for making the create to-do list input field accessible separately.
  let input;
  // Prepare the window location hash for being reactive.
  $: windowLocationHash = undefined;
  // Make the count of all to-do lists reactive.
  $: countToDoListsAll = toDoLists.length;
  // Make the count of pending and done to-do lists reactive.
  let countToDoListsPending;
  let countToDoListsDone;
  $: if (toDoLists.length > 0) {
    const toDoListsPending = toDoLists.filter(toDoList => {
      return toDoList.done !== true;
    });
    const toDoListsDone = toDoLists.filter(toDoList => {
      return toDoList.done !== false;
    });
    countToDoListsPending = toDoListsPending.length;
    countToDoListsDone = toDoListsDone.length;
  }
  onMount(() => {
    // Determine the window location hash.
    windowLocationHash = window.location.hash === '' ? '#all' : window.location.hash;
    // Add an event handler for when the window hash changed.
    window.addEventListener('hashchange', handleHashChange);
    // Add an event handler for when the window mouse moves (and to-do lists' links are hovered).
    window.addEventListener('mousemove', handleMouseMove);
    input.focus();
  });
  onDestroy(() => {
    // Remove the event handler for when the window hash changed.
    window.removeEventListener('hashchange', handleHashChange);
    // Remove the event handler for when the window mouse moves (and to-do lists' links are hovered).
    window.removeEventListener('mousemove', handleMouseMove);
  });
  /**
   * Method to store to-do lists to local storage.
   * @param {object} toDoLists - The to-do lists to store.
   */
  function store(toDoLists) {
    localStorage.setItem('toDoLists', JSON.stringify(toDoLists));
  }
  /**
   * Method to create a new to-do list.
   * @param {object} event - The event object containing the new to-do list's name.
   */
  function createToDoList(event) {
    if (event.key === 'Enter') {
      if (!event.target.value.match(/^\s*$/)) {
        const toDoListDateCreated = new Date();
        const toDoList = {
          id: toDoListDateCreated.getTime(),
          created: toDoListDateCreated.toJSON(),
          lastRenamed: null,
          lastUpdated: null,
          name: input.value,
          done: false,
          items: []
        };
        toDoLists = [...toDoLists, toDoList];
        store(toDoLists);
        event.target.value = '';
        event.target.focus();
      }
    }
  }
  /**
   * Method to handle a to-do list update.
   * @param {object} nodeEditToDoList - The edit button element node.
   * @param {object} nodeToDoListName - The to-do list's name text node.
   * @param {object} nodeInputUpdateToDoList - The update input field element node.
   */
  function handleToDoListUpdate(nodeEditToDoList, nodeToDoListName, nodeInputUpdateToDoList) {
    // Only accept input that isn't all whitespace.
    if (!nodeInputUpdateToDoList.value.match(/^\s*$/)) {
      if (nodeInputUpdateToDoList.value !== nodeToDoListName.textContent) {
        toDoLists = toDoLists.map(toDoList => {
          if (toDoList.id === +nodeInputUpdateToDoList.parentNode.id) {
            const toDoListDateLastRenamed = new Date();
            toDoList.lastRenamed = toDoListDateLastRenamed.toJSON();
            toDoList.name = nodeInputUpdateToDoList.value;
          }
          return {...toDoList};
        });
      }
      nodeInputUpdateToDoList.remove();
      if (nodeEditToDoList.parentNode.querySelector('label.name')) {
        nodeEditToDoList.parentNode.querySelector('label.name').classList.remove('hidden');
      }
      store(toDoLists);
      input.focus();
    }
  }
  /**
   * Method to update a to-do list.
   * @param {object} event - The event object containing the edit button element node.
   */
  function updateToDoList(event) {
    const nodeEditToDoList = event.target;
    const nodeToDoListName = nodeEditToDoList.parentNode.querySelector('label.name a');
    const nodeInputUpdateToDoList = document.createElement('input');
    nodeInputUpdateToDoList.style = 'width: calc(100% - 1.5em)';
    nodeInputUpdateToDoList.type = 'text';
    nodeInputUpdateToDoList.value = nodeToDoListName.textContent;
    nodeEditToDoList.parentNode.appendChild(nodeInputUpdateToDoList);
    if (nodeEditToDoList.parentNode.querySelector('label.name')) {
      nodeEditToDoList.parentNode.querySelector('label.name').classList.add('hidden');
    }
    nodeInputUpdateToDoList.focus();
    nodeInputUpdateToDoList.addEventListener('blur', event => {
      handleToDoListUpdate(nodeEditToDoList, nodeToDoListName, nodeInputUpdateToDoList);
    });
    nodeInputUpdateToDoList.addEventListener('keydown', event => {
      if (event.key === 'Enter') {
        handleToDoListUpdate(nodeEditToDoList, nodeToDoListName, nodeInputUpdateToDoList);
      } else if (event.key === 'Escape') {
        nodeInputUpdateToDoList.remove();
        if (nodeEditToDoList.parentNode.querySelector('label.name')) {
          nodeEditToDoList.parentNode.querySelector('label.name').classList.remove('hidden');
        }
      }
    });
  }
  /**
   * Method to delete a to-do list.
   * @param {number} id - The to-do list's ID.
   */
  function deleteToDoList(id) {
    toDoLists = toDoLists.filter(toDoList => {
      return toDoList.id !== id;
    });
    store(toDoLists);
  }
  /**
   * Method to toggle a to-do list.
   * @param {number} id - The to-do list's ID.
   */
  function toggleToDoList(id) {
    toDoLists = toDoLists.map(toDoList => {
      if (toDoList.id === id) {
        toDoList.done = !toDoList.done;
      }
      return {...toDoList};
    });
    store(toDoLists);
  }
  /**
   * Method to toggle all to-do lists.
   */
  function toggleAllToDoLists() {
    toDoLists = toDoLists.map(toDoList => {
      toDoList.done = !toDoList.done;
      return {...toDoList};
    });
    store(toDoLists);
  }
  /**
   * Method to clear (delete all done) to-do lists (to-do lists).
   */
  function clearToDoLists(event) {
    event.target.classList.add('hidden');
    event.target.nextElementSibling.classList.remove('hidden');
  }
  /**
   * Method to confirm the clearance (deletion of all done) of the to-do lists (to-do lists).
   */
  function clearToDoListsYes(event) {
    event.target.parentNode.classList.add('hidden');
    toDoLists = toDoLists.filter(toDoList => {
      return toDoList.done === false;
    });
    store(toDoLists);
    input.focus();
  }
  /**
   * Method to reject the clearance (deletion of all done) of the to-do lists (to-do lists).
   */
  function clearToDoListsNo(event) {
    event.target.parentNode.classList.add('hidden');
    event.target.parentNode.previousElementSibling.classList.remove('hidden');
  }
  /**
   * Method to handle the window hash change event.
   */
  const handleHashChange = () => {
    windowLocationHash = window.location.hash;
  }
  /**
   * Method to handle the window mouse move event for when the to-do lists' links are hovered.
   */
  const handleMouseMove = (event) => {
    const previewTooltips = container.querySelectorAll('section.tooltip');
    previewTooltips.forEach(previewTooltip => {
      previewTooltip.style.top = (window.innerHeight - event.clientY < previewTooltip.offsetHeight ? event.clientY - previewTooltip.offsetHeight : event.clientY) + 'px';
      previewTooltip.style.left = event.clientX + 'px';
    });
  }
  /**
   * Method to route the to-do lists.
   */
  function routeToDoList(event) {
    // Prevent a page reload.
    event.preventDefault();
    // Assign the hyperlink's target URL to a pathname object.
    const {pathname: path} = new URL(event.target.href);
    // Add a new entry to the browser's session history stack.
    window.history.pushState({path}, '', path);
    // Assign a new popstate event including the previously assigned pathname object and trigger it manually.
    const popStateEvent = new PopStateEvent('popstate', {pathname: path});
    dispatchEvent(popStateEvent);
  }
  /**
   * Method to set the ID attribute of the to-do list preview component.
   */
  function setAttributeId(node, id) {
    node.setAttribute('id', id);
  }
</script>

<section class="container" bind:this={container}>
  <header>
    <h1>t<span id="first-o">o</span>-d<span id="second-o">o</span> lists</h1>
    <h2>Overview of your to-do lists</h2>
    <button on:click={toggleAllToDoLists} id="toggle-all" class="{countToDoListsAll > 0 ? '' : 'hide'}" title="Click to toggle all"></button>
    <input on:keydown={createToDoList} bind:this={input} id="create" class="{countToDoListsAll > 0 ? 'shrink' : ''}" type="text" placeholder="What do you have to do?">
  </header>
  <section class="to-do-lists">
    {#if countToDoListsAll > 0}
    <ul class="to-do-lists">
      {#each toDoLists as toDoList}
      {#if (windowLocationHash === '#all') || (windowLocationHash === '#pending' && !toDoList.done) || (windowLocationHash === '#done' && toDoList.done)}
      <li id="{toDoList.id}" class="{toDoList.done ? 'done' : 'pending'}">
        <input on:click={toggleToDoList(toDoList.id)} class="select" title="Click to mark as {toDoList.done === false ? 'done' : 'pending'}" type="checkbox" bind:checked={toDoList.done}><!--
     --><label on:click={updateToDoList} data-created="{toDoList.created}" data-last-renamed="{toDoList.lastRenamed}" data-last-modified="{toDoList.lastUpdated}" class="name">
          <a on:click={routeToDoList} href="/svelte-to-do-list/to-do-list/{toDoList.id}">{toDoList.name}</a>
          <span class="count">({toDoList.items.length} {toDoList.items.length === 1 ? 'item' : 'items'}{toDoList.items.length > 0 ? ': ' + toDoList.items.filter(toDoListItem => {return toDoListItem.done !== true}).length + ' pending, ' + toDoList.items.filter(toDoListItem => {return toDoListItem.done !== false}).length + ' done' : ''})</span>
          <section class="tooltip">
            <svelte-to-do-list-preview id="{toDoList.id}" use:setAttributeId={toDoList.id} todolist={toDoList}></svelte-to-do-list-preview>
          </section>
        </label><!--
     --><button on:click={updateToDoList} class="edit" title="Click to edit"></button><!--
     --><button on:click={deleteToDoList(toDoList.id)} class="delete" title="Click to delete"></button><!--
   --></li>
      {/if}
      {/each}
    </ul>
    {:else}
    <p>There are no to-do lists to show!</p>
    {/if}
  </section>
  <hr class="{countToDoListsAll > 0 ? '' : 'hidden'}">
  <footer class="{countToDoListsAll > 0 ? '' : 'hidden'}">
    <span id="count" class="{countToDoListsAll > 0 ? '' : 'hide hidden'}">{countToDoListsAll} ({countToDoListsPending}/{countToDoListsDone})</span>
    <ul class="filters">
      <li><a id="all" class="{windowLocationHash === '#all' ? 'selected' : ''}" href="#all">All</a></li>
      <li><a id="pending" class="{windowLocationHash === '#pending' ? 'selected' : ''}" href="#pending">Pending</a></li>
      <li><a id="done" class="{windowLocationHash === '#done' ? 'selected' : ''}" href="#done">Done</a></li>
    </ul>
    <button on:click={clearToDoLists} id="clear" class="{countToDoListsDone > 0 ? '' : 'hide hidden'}" title="Click to delete done">Clear</button>
    <span id="confirmation" class="hidden">
      Sure?
      <button on:click={clearToDoListsYes} id="yes" title="Click to confirm">Yes</button>
      <button on:click={clearToDoListsNo} id="no" title="Click to reject">No</button>
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
  section.to-do-lists ul.to-do-lists {
    padding: 0;
    text-align: start;
    list-style: none;
  }
  section.to-do-lists ul.to-do-lists li {
    position: relative;
    transition: opacity var(--long) ease-in-out, outline-color var(--short) ease-in-out;
    padding: 0.5em;
    outline: transparent dashed 0.1em;
    border-radius: 0.5em;
  }
  section.to-do-lists ul.to-do-lists li:hover {
    outline: var(--gray-semi-transparent) dashed 0.1em;
  }
  section.to-do-lists ul.to-do-lists li input[type=checkbox].select {
    appearance: none;
    position: relative;
    bottom: 0.1em;
    transition: background var(--long) ease-in-out, border var(--long) ease-in-out;
    margin: 0 0.5em 0 0;
    cursor: pointer;
    font-size: inherit;
    vertical-align: middle;
  }
  section.to-do-lists ul.to-do-lists li.pending input[type=checkbox].select,
  section.to-do-lists ul.to-do-lists li.done input[type=checkbox].select {
    display: inline-block;
    border-radius: 50%;
    box-sizing: border-box;
    width: 1em;
    height: 1em;
  }
  section.to-do-lists ul.to-do-lists li.pending input[type=checkbox].select {
    border: 0.2em solid var(--cyan);
    background: transparent;
  }
  section.to-do-lists ul.to-do-lists li.done input[type=checkbox].select {
    border: 0.5em solid var(--cyan);
    background: var(--cyan);
  }
  section.to-do-lists ul.to-do-lists li label.name {
    --strikethrough: 0;
    transition: background-size var(--long) ease-in-out, color var(--long) ease-in-out;
    background: linear-gradient(to right, transparent 0, currentcolor 0) no-repeat right center / calc(var(--strikethrough) * 100%) 0.1em;
    background-position-x: left;
    cursor: text;
    word-break: break-all;
  }
  section.to-do-lists ul.to-do-lists li label.name a {
    text-decoration: none;
    color: var(--cyan);
  }
  section.to-do-lists ul.to-do-lists li.done label.name {
    --strikethrough: 1;
    color: var(--gray);
  }
  section.to-do-lists ul.to-do-lists li button.edit {
    all: unset;
    position: absolute;
    bottom: 0.6em;
    right: 1.5em;
    justify-content: space-around;
    display: flex;
    opacity: 0;
    transform: rotate(45deg);
    transition: opacity var(--short) ease-in-out;
    margin: 0 0.5em 0 0;
    width: 1em;
    height: 1em;
    cursor: pointer;
  }
  section.to-do-lists ul.to-do-lists li button.edit::before,
  section.to-do-lists ul.to-do-lists li button.edit::after {
    content: '';
    position: absolute;
    border-radius: 0.05em;
    width: 0.25em;
  }
  section.to-do-lists ul.to-do-lists li button.edit::before {
    height: 0.7em;
    background: var(--cyan);
  }
  section.to-do-lists ul.to-do-lists li button.edit::after {
    bottom: 0;
    width: 0;
    height: 0;
    border-left: 0.125em solid transparent;
    border-right: 0.125em solid transparent;
    border-top: 0.25em solid var(--cyan);
  }
  section.to-do-lists ul.to-do-lists li:hover button.edit {
    opacity: 1;
  }
  section.to-do-lists ul.to-do-lists li button.delete {
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
  section.to-do-lists ul.to-do-lists li button.delete::before,
  section.to-do-lists ul.to-do-lists li button.delete::after {
    content: '';
    position: absolute;
    border-radius: 0.1em;
    width: 0.2em;
    height: 1em;
    background: var(--cyan);
  }
  section.to-do-lists ul.to-do-lists li button.delete::before {
    transform: rotate(45deg);
  }
  section.to-do-lists ul.to-do-lists li button.delete::after {
    transform: rotate(-45deg);
  }
  section.to-do-lists ul.to-do-lists li:hover button.delete {
    opacity: 1;
  }
  section.to-do-lists ul.to-do-lists li input[type=text] {
    width: calc(100% - 1.5em);
  }
  section.to-do-lists ul.to-do-lists li label.name section.tooltip {
    position: relative;
    z-index: 1;
    display: none;
    border: 0.15em solid var(--cyan);
    border-radius: 0.5em;
    box-sizing: border-box;
    width: 22.5vw;
    padding: 0 1em;
    min-width: 22.5em;
    background: white;
  }
  section.to-do-lists ul.to-do-lists li label.name a:hover ~ section.tooltip {
    position: fixed;
    display: block;
    overflow: hidden;
  }
  section.to-do-lists ul.to-do-lists li.done label.name section.tooltip svelte-to-do-list-preview {
    opacity: 50%;
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
