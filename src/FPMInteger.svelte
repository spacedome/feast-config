<script>
  export let value;
  export let fpmIndex;
  export let description;
  export let min = "";
  export let max = "";
  export let disabled = null;
</script>

<div class="columns is-mobile">
  <div class="column is-2">
    <div class="field">
      <div class="field">
        <div class="control">
          <input
            type="number"
            class="input"
            {disabled}
            bind:value
            {min}
            {max}
          />
        </div>
      </div>
    </div>
  </div>
  <div class="column is-small is-2">
    <p
      class={"box is-family-code has-text-centered is-size-6" +
        (disabled ? " disabled" : "")}
    >
      fpm({fpmIndex})={value}
    </p>
  </div>
  <div class="column">
    <p class={"box" + (disabled ? " disabled" : "")}>{@html description}</p>
  </div>
</div>
