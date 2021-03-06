<script>
    export let dataType='d';
    export let real=true;
    export let probType='ev';
    export let expert=false;
    export let storageFormat='dense';

    $: dtype = (dataType === 'd' ? "double" : "dcomplex");
    $: E = (real ? "double" : "dcomplex") + "(M0)";
    $: X = (real ? "double" : "dcomplex") + (probType === 'gv' ? "(N,2*M0)" : "(N,M0)");
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
    $: zwork1 = "dcomplex" + (probType === 'gv' ? "(2*M0)" : "(M0)");
    $: list_rci = [
        ["ijob", "integer", "in/out", "ID of teh FEAST_RCI operation\nOn entry: ijob=-1 (initialization)"],
        ["N", "integer", "in", "Size of the system"],
        ["Ze", "complex", "out", "Coordinate along the complex contour"],
        ["work1", "double(N,M0)", "in/out", "Workspace"],
        ["zwork1", zwork1, "in/out", "Workspace"],
        ["zwork2", "dcomplex(N,M0)", "in/out", "Workspace"],
        ["Aq,Bq", "double(M0,M0)", "in/out", "Workspace for reduced problem"],
        ["zAq,zBq", "dcomplex(M0,M0)", "in/out", "Workspace for reduced problem"],
    ]    
    $: list_banded = [
        ["UPLO", "character", "in", "Matrix Storage: 'F': Full, 'L': Lower, 'U': Upper"],
        ["N", "integer", "in", "Size of the system"],
        ["A", dim_a, "in", "Matrix"],
        ["B", dim_b, "in", "Matrix", 'gv'],
        ["kla", "integer", "in", "Number of subdiagonals within band of A"],
        ["klu", "integer", "in", "Number of superdiagonals within band of A", 'gv'],
        ["klb", "integer", "in", "Number of subdiagonals within band of B", 'gv'],
        ["kub", "integer", "in", "Number of superdiagonals within band of B", 'gv'],
        ["LDA", "integer", "in", "Leading dimension of A"],
        ["LDB", "integer", "in", "Leading dimension of B", 'gv'],
    ]
    $: list_sparse = [
        ["UPLO", "character", "in", "Matrix Storage: 'F': Full, 'L': Lower, 'U': Upper"],
        ["N", "integer", "in", "Size of the system"],
        ["A", dim_a, "in", "Matrix"],
        ["B", dim_b, "in", "Matrix", 'gv'],
        ["IA", "integer(N+1)", "in", "Sparse CSR Row array of A"],
        ["JA", "integer(IA(N+1)-1)", "in", "Sparse CSR Column array of A"],
        ["IB", "integer(N+1)", "in", "Sparse CSR Row array of B", 'gv'],
        ["JB", "integer(IB(N+1)-1)", "in", "Sparse CSR Column array of B", 'gv'],
    ]
    $: dim_a = dtype + (storageFormat === 'rci' ? '' : (storageFormat === 'sparse' ? '(IA(N+1)-1)' : '(LDA,N)'));
    $: dim_b = dtype + (storageFormat === 'rci' ? '' : (storageFormat === 'sparse' ? '(IB(N+1)-1)' : '(LDB,N)'));
    $: list_dense = [
        ["UPLO", "character", "in", "Matrix Storage: 'F': Full, 'L': Lower, 'U': Upper"],
        ["N", "integer", "in", "Size of the system"],
        ["A", dim_a, "in", "Matrix"],
        ["B", dim_b, "in", "Matrix", 'gv'],
        ["LDA", "integer", "in", "Leading dimension of A"],
        ["LDB", "integer", "in", "Leading dimension of B", 'gv'],
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
        {#if (storageFormat === 'rci')}
            {#each list_rci as parameter}
                <tr>
                    <td class="is-family-code has-text-weight-medium">{parameter[0]}</td>
                    <td class="is-family-code">{parameter[1]}</td>
                    <td>{parameter[2]}</td>
                    <td>{parameter[3]}</td>
                </tr>
            {/each}
        {:else if (storageFormat === 'dense') }
            {#each list_dense as parameter}
                {#if (probType === 'gv' || parameter[4] !== 'gv')}
                    <tr>
                        <td class="is-family-code has-text-weight-medium">{parameter[0]}</td>
                        <td class="is-family-code">{parameter[1]}</td>
                        <td>{parameter[2]}</td>
                        <td>{parameter[3]}</td>
                    </tr>
                {/if}
            {/each}
        {:else if (storageFormat === 'sparse') }
            {#each list_sparse as parameter}
                {#if (probType === 'gv' || parameter[4] !== 'gv')}
                    <tr>
                        <td class="is-family-code has-text-weight-medium">{parameter[0]}</td>
                        <td class="is-family-code">{parameter[1]}</td>
                        <td>{parameter[2]}</td>
                        <td>{parameter[3]}</td>
                    </tr>
                {/if}
            {/each}
        {:else if (storageFormat === 'banded') }
            {#each list_banded as parameter}
                {#if (probType === 'gv' || parameter[4] !== 'gv')}
                    <tr>
                        <td class="is-family-code has-text-weight-medium">{parameter[0]}</td>
                        <td class="is-family-code">{parameter[1]}</td>
                        <td>{parameter[2]}</td>
                        <td>{parameter[3]}</td>
                    </tr>
                {/if}
            {/each}
        {/if}
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