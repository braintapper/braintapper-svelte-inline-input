<script>
  
  import { tick, createEventDispatcher } from 'svelte'

  const dispatch = createEventDispatcher();

  export let style = undefined;
  export let inputStyle = undefined;
  export let type = 'text';
  export let placeholder = '';
  export let labelClasses = '';
  export let inputClasses = '';

  // this control is for an item with an _id/id (depends on db type)
  
  export let item = { _id: "", name: "" };
  export let inputAttribute = "name";
  export let idAttribute = "_id";


  export let doubleClick = true;

  export let id = undefined;
  export let delay = 250;

  export let editing = false;

  // todo: export let selectAllOnEdit = false;
  // todo: export let forceEditMode = false;

  let inputEl = undefined
  let label = undefined
  let originalValue = undefined

  let value = undefined


  // Computed
  // todo: clean these up

  $: isText = type === 'text';
  $: isNumber = type === 'number';
  $: if (isNumber) {
        label = value === '' ? placeholder : value;
      } else if (isText) {
        label = value ? value : placeholder;
      }
  const toggle = async (_) => {
    editing = !editing;

    originalValue = value;
    if (editing) {
      await tick();
      inputEl.focus();
      inputEl.select();
      value = "" + item[inputAttribute];

    }
  }



  let handleInput = (e) => {
    if (isNumber) {
      value = +e.target.value
    } else {
      value = e.target.value
    }
  }


  let handleKeyup = (e) => {
    dispatch("keyup", value);
    switch (e.keyCode) {
      case 13:
        inputEl.blur();
        unchanged = (value == item[idAttribute]);
        if (!(unchanged && (value.length > 0))) {
          eventObject = {
            _id: item[idAttribute]
          }
          eventObject[inputAttribute] = value;
          dispatch("update",eventObject);
        }   
        break;
      case 27:
        value = originalValue;
        inputEl.blur();
        break;
    }
  }

  let handleBlur = (_) => {
    toggle()
  }


  // Click tracking

  let clickCount = 0

  clickReset = ()=> {
    switch (clickCount) {
      case 1:
        single()
        break;
      case 2:
        double()
        break;
      default:
        clickCount = 0;
    }
  }

  let delayedReset = clickReset.debounce(delay)

  let single = () => {
    clickCount = 0;
    if (!doubleClick) {
      toggle()
    }
    dispatch("clicked", id);
  }

  let double = () => {
    clickCount = 0;
    if (doubleClick) {
      toggle();
    }
    dispatch("doubleclicked", id);
  }

  let click = ()=> {
    clickCount++;
    delayedReset();
  }



</script>

{#if editing && (isText || isNumber)}
  <input
    class={inputClasses}
    style={inputStyle}
    bind:this={inputEl}
    {type}
    {value}
    {placeholder}
    on:click|stopPropagation
    on:input={handleInput}
    on:keyup={handleKeyup}
    on:blur={handleBlur}>
{:else}
  <div
    {id}
    class={labelClasses}
    {style}>
    <span on:click={click}><slot><span>{item[inputAttribute]}<span spacer/></span></slot></span>
  </div>
{/if}


<style>

  input, div, div span {
    font-family: inherit;
    font-size: inherit;
    font-weight: inherit;
    border: 0px solid transparent;
    padding: 0px; }

  input {
    width: 100%;
    color: var(--dark-red); }

  input::selection {
    background: var(--dark-red);
    color: white; }

  div {
    min-height: var(--font-xl);
    border: 1px solid transparent; }


</style>