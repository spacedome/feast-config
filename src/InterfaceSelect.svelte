<script>
    export let value;
    export let description;
    export let state="";
    export let disabled=null;
</script>

<div class="columns is-mobile">
    <div class="column is-one-quarter">
        <div class="field">
            <p class="control is-expanded">
                <span class={"select is-fullwidth" + (disabled ? " disabled " : " ") + state} >
                    <select bind:value={value} disabled={disabled} on:change>
                        <slot></slot>
                    </select>
                </span>
            </p>
        </div>
    </div>
    <div class="column">
        <p class={"box" + (disabled ? " disabled" : "")}>{description}</p>
    </div>
</div>
