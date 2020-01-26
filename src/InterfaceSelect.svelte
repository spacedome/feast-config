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
                <span class={"select is-fullwidth " + state} >
                    <select bind:value={value} disabled={disabled}>
                        <slot></slot>
                    </select>
                </span>
            </p>
        </div>
    </div>
    <div class="column">
        <p class="box">{description}</p>
    </div>
</div>