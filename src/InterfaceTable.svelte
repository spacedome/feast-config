<script>
    export let dataType='d';
    export let real=true;
    export let probType='ev';
    export let expert=false;

    $: dtype = (dataType === 'd' ? "double" : "complex");
    $: E = (real ? "double" : "complex") + "(M0)";
    $: X = (real ? "double" : "complex") + (probType === 'gv' ? "(N,2*M0)" : "(N,M0)");
    $: res = "double" + (probType === 'gv' ? "(2*M0)" : "(M0)");
    $: common = [
        ["fpm", "integer(64)", "in", "FEAST input parameters (size at least 64)"],
        ["epsout", "double", "out", "Relative error on the trace"],
        ["loop", "integer", "out", "Number of FEAST subspace iterations"],
        ["M0", "integer", "in/out", "Search subspace dimension \nin: initial guess, out: new suitable M0 if guess too large"],
        ["E", E, "out", "Eigenvalues\nThe first M values are in the search interval\nThe remaining M0-M are outside"],
        ["X", X, "in/out", "Eigenvectors (N = size of system) - same ordering as E\nin: guess subspace if fpm(5)=1\nout: (right) eigenvector solutions X(1:N, 1:M)\n note: if calculated, left eigenvalues in X(1:N, M0+1:M0+M)"],
        ["M", "integer", "out", "Number of eigenvalues found in search interval"],
        ["res", res, "out", "Relative residual"],
        ["info", "integer", "out", "Error handling: info=0 indicates successful exit"],
    ]
</script>

<table class="table is-striped is-narrow is-hoverable is-fullwidth">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th><abbr title="Input/Output">I/O</abbr></th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        {#each common as parameter}
            <tr>
                <td class="is-family-code has-text-weight-medium">{parameter[0]}</td>
                <td class="is-family-code">{parameter[1]}</td>
                <td>{parameter[2]}</td>
                <td>{parameter[3]}</td>
            </tr>
        {/each}
        {#if expert}
            <tr>
                <td class="is-family-code has-text-weight-medium">Zne,Wne</td>
                <td class="is-family-code">complex(n)</td>
                <td>in</td>
                <td>Custom integration nodes and weights (for n nodes)</td>
            </tr>
        {/if}
    </tbody>
</table>

<style>
table {
    margin-top: 0.5em;
}
td {
    white-space: pre-line;
}
</style>