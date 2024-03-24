<script>
  export let value;
  export let fpmIndex;
  export let description;
  export let state = "";
  export let nonum = false;
  export let disabled = null;
</script>

<div class="columns is-mobile">
  <div class="column is-2">
    <div class="field">
      <p class="control is-expanded">
        <span
          class={"select is-fullwidth " +
            (disabled ? " disabled " : " ") +
            state}
        >
          <select bind:value {disabled} on:change>
            <slot />>
          </select>
        </span>
      </p>
    </div>
  </div>
  <div class="column is-small is-2">
    <p
      class={"box is-family-code has-text-centered is-size-6" +
        (disabled ? " disabled" : "")}
    >
      fpm({fpmIndex})={nonum ? "x" : value}
    </p>
  </div>
  <div class="column">
    <p class={"box" + (disabled ? " disabled" : "")}>{description}</p>
  </div>
</div>
