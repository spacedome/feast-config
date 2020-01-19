<script>

	export let name;

	let params = {
		version:	3,
		problem_type: 's', // s: symmetric, h: hermitian, g: general
		data_type: 'd', // s: single, d: double, c: comp single, z: comp double
		storage_format: 'dense', // dense, sparse, banded, rci
		generalized: false,
	}

	let fpm_default = { //given values are feastinit defaults
		1: 0,  // Print runtime comments on screen, 0: no, 1: yes
		2: 8,  // number contour points for Hermitian (half-contour), if fpm(16=0,2) can be {1 to 20, 24, 32, 40, 48, 56}, if fpm(16)=1 all values permitted
		3: 12, // stopping convergence criteria for double prec (10^(-x))
		4: 20, // max number of FEAST loops
		5: 0,  // provide inital guess subspace, 0: no, 1: yes
		6: 1,  // conv critera for eigpair in search interval, 0: rel err on trace epsout, 1: rel residual max(res)
		7: 5,  // stopping convergence criteria for single prec (10^(-x))
		8: 16, // number contour points for non-Hermitian (full-contour), if fpm(17)=0 can be {2 to 40, 48, 64, 80, 96, 112}, if fpm(17)=1 all values permitted
		9: -1, // user defined MPI communicator (?)
		10: 0, // store factorizations with predefined interfaces 0: no, 1 : yes
		16: 0, // hermitian integration type, 0: gauss, 1:trapezoid, 2: zolotarev
		17: 1, // non-hermitian integration type, 0: gauss, 1:trapezoid
	}

	let fpm = JSON.parse(JSON.stringify(fpm_default)); // deep clone

	let feast_arg = "\{List\}, fpm, epsout, loop, \{List-I\}, M0, E, X, M, res, info";

	function is_hermitian(parameters) {
		return (parameters.problem_type === 's' || parameters.problem_type === 'h');
	}

	function is_single_prec(parameters) {
		return (parameters.data_type === 's' || parameters.data_type === 'c');
	}

	function feast_call(parameters) {
		let T = 'd';
		let Y = 's';
		let F = 'f';
		let G = (parameters.generalized ? 'gv' : 'ev' );
		switch (parameters.data_type) {
			case 's': // single
			case 'd': // double
			case 'c': // complex single
			case 'z': // complex double
				T = parameters.data_type;
				break;
			default:
				throw 'Error';
		}
		switch (parameters.problem_type) {
			case 's': // symmetric
			case 'h': // hermitian
			case 'g': // general
				Y = parameters.problem_type;
				break;
			default:
				throw 'Error';
		}
		switch (parameters.storage_format) {
			case 'dense':
				F = (Y === 's' ? 'y' : 'e');
				break;
			case 'rci':
				F = 'rci';
				G = '';
				break;
			case 'banded':
				F = 'b';
				break;
			case 'sparse':
				F = 'csr';
				break;
			default:
				throw 'Error';
		}
		return T + "feast_" + Y + F + G;
	}

	function input_params(feastparams, feastparams_default) {
		let fortran = "integer, dimension(64) :: fpm\ncall feastinit(fpm)\n";
		let count = 2;
		for (let key in feastparams) {
		  if (!(feastparams[key] === feastparams_default[key])) {
				fortran += "fpm(" + key + ") = " + feastparams[key] + "\n";
				count +=1
			}
		}
		return {count: count, text: fortran};
	}

	$: param_diff = input_params(fpm, fpm_default);

</script>

<main class="section">
	<div class="container">
		<div class="columns is-mobile">
			<div class="column">
				<h1 class="title is-1">
				  {name}
				</h1>
				<p class="subtitle is-4">
				  An interface for generating FEAST parameters
				</p>
				<p class="content is-marginless">
					See the <a href="http://www.ecs.umass.edu/~polizzi/feast/">homepage</a> of the FEAST eigenvalue solver for details and documentation.
				</p>
				<p class="content">
					Configure FEAST interface before input parameters.
				</p>
			</div>
			<div class="column is-2">
				<figure class="image is-square logo">
				  <img src="/FEAST.svg" alt="FEAST">
				</figure>
			</div>
		</div>
		<hr/>
	</div>



	<div class="container">

		<h2 class="title is-4">
			FEAST Interfaces
		</h2>

		<div class="columns is-mobile"> <!-- version -->
			<div class="column is-one-quarter">
				<div class="field">
					<p class="control is-expanded">
						<span class="select is-fullwidth">
							<select bind:value="{params.version}">
								<!-- <option value={4}>FEAST 4.0</option> -->
								<option value={3}>FEAST 3.0</option>
								<option value={2}>FEAST 2.1</option>
							</select>
						</span>
					</p>
				</div>
			</div>
			<div class="column">
				<p class="box">FEAST version. Parameters are forward compatible. v2.1 is used in MKL</p>
			</div>
		</div>

		<div class="columns is-mobile"> <!-- data type -->
			<div class="column is-one-quarter">
				<div class="field">
					<p class="control is-expanded">
						<span class="select is-fullwidth">
							<select bind:value="{params.data_type}">
								<option value="{'d'}">Double Precision (d)</option>
								<option value="{'s'}">Single Precision (s)</option>
								<option value="{'z'}">Complex Double (z)</option>
								<option value="{'c'}">Complex Single (c)</option>
							</select>
						</span>
					</p>
				</div>
			</div>
			<div class="column">
				<p class="box">Matrix data type</p>
			</div>
		</div>

		<div class="columns is-mobile"> <!-- problem type -->
			<div class="column is-one-quarter">
				<div class="field">
					<p class="control is-expanded">
						<span class="select is-fullwidth">
							<select bind:value="{params.problem_type}">
								<option value="{'s'}">Symmetric (s)</option>
								<option value="{'h'}">Hermitian (h)</option>
								<option value="{'g'}">General (g)</option>
							</select>
						</span>
					</p>
				</div>
			</div>
			<div class="column">
				<p class="box">Type/symmetry of problem</p>
			</div>
		</div>

		<div class="columns is-mobile"> <!-- storage format -->
			<div class="column is-one-quarter">
				<div class="field">
					<p class="control is-expanded">
						<span class="select is-fullwidth">
							<select bind:value="{params.storage_format}">
								<option value="{'dense'}">Dense</option>
								<option value="{'banded'}">Banded</option>
								<option value="{'sparse'}">Sparse</option>
								<option value="{'rci'}">RCI</option>
							</select>
						</span>
					</p>
				</div>
			</div>
			<div class="column">
				<p class="box">Matrix storage format or Reverse Communication Interface (RCI)</p>
			</div>
		</div>

		<div class="columns">
			<div class="column control">
	  		<textarea class="textarea has-fixed-size is-family-code" rows="1" readonly>{feast_call(params)}</textarea>
			</div>
		</div>

	</div>

	<div class="container">

		<h2 class="title is-4">
			FEAST Input Parameters (<span class="is-family-code">fpm</span>)
		</h2>

		<div class="columns is-mobile"> <!-- fpm 1 -->
			<div class="column is-2">
				<div class="field">
					<p class="control is-expanded">
						<span class="select is-fullwidth">
							<select bind:value="{fpm[1]}">
								<option value={0}>No</option>
								<option value={1}>Yes</option>
							</select>
						</span>
					</p>
				</div>
			</div>
			<div class="column is-small is-2">
				<p class="box is-family-code has-text-centered is-size-6">fpm(1) = {fpm[1]}</p>
			</div>
			<div class="column">
				<p class="box">Print runtime comments on screen</p>
			</div>
		</div>

		{#if is_hermitian(params)}
			<div class="columns is-mobile"> <!-- fpm 2 -->
				<div class="column is-2">
					{#if fpm[16]===1}
						<div class="field">
							<div class="control">
								<input type=number class="input" bind:value="{fpm[2]}" min=1 max=65536>
							</div>
						</div>
					{:else}
						<div class="field">
							<p class="control is-expanded">
								<span class="select is-fullwidth">
									<select bind:value="{fpm[2]}">
										{#each [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,24,32,40,48,56] as points}
										<option value={points}>{points}</option>
										{/each}
									</select>
								</span>
							</p>
						</div>
					{/if}
				</div>
				<div class="column is-small is-2">
					<p class="box is-family-code has-text-centered is-size-6">fpm(2) = {fpm[2]}</p>
				</div>
				<div class="column">
					<p class="box"> Number of contour points (Hermitian only, half contour)</p>
				</div>
			</div>
		{/if}

		{#if !is_single_prec(params)}
		<div class="columns is-mobile"> <!-- fpm 3 -->
			<div class="column is-2">
				<div class="field">
					<div class="control">
						<input type=number class="input" bind:value="{fpm[3]}">
					</div>
				</div>
			</div>
			<div class="column is-small is-2">
				<p class="box is-family-code has-text-centered is-size-6">fpm(3) = {fpm[3]}</p>
			</div>
			<div class="column">
				<p class="box">Stopping convergence criteria for double precision (&epsilon = <var>10<sup>-<i>x</i></sup></var>)</p>
			</div>
		</div>
		{/if}

		<div class="columns is-mobile"> <!-- fpm 4 -->
			<div class="column is-2">
				<div class="field">
					<div class="control">
						<input type=number class="input" bind:value="{fpm[4]}" min=0>
					</div>
				</div>
			</div>
			<div class="column is-small is-2">
				<p class="box is-family-code has-text-centered is-size-6">fpm(4) = {fpm[4]}</p>
			</div>
			<div class="column">
				<p class="box">Maximum number of FEAT refinement iterations</p>
			</div>
		</div>

		<div class="columns is-mobile"> <!-- fpm 5 -->
			<div class="column is-2">
				<div class="field">
					<p class="control is-expanded">
						<span class="select is-fullwidth">
							<select bind:value="{fpm[5]}">
								<option value={0}>No</option>
								<option value={1}>Yes</option>
							</select>
						</span>
					</p>
				</div>
			</div>
			<div class="column is-small is-2">
				<p class="box is-family-code has-text-centered is-size-6">fpm(5) = {fpm[5]}</p>
			</div>
			<div class="column">
				<p class="box">Provide initial guess subspace</p>
			</div>
		</div>

		<div class="columns is-mobile"> <!-- fpm 6 -->
			<div class="column is-2">
				<div class="field">
					<p class="control is-expanded">
						<span class="select is-fullwidth">
							<select bind:value="{fpm[6]}">
								<option value={1}>Relative Residual</option>
								<option value={0}>Relative Trace</option>
							</select>
						</span>
					</p>
				</div>
			</div>
			<div class="column is-small is-2">
				<p class="box is-family-code has-text-centered is-size-6">fpm(6) = {fpm[6]}</p>
			</div>
			<div class="column">
				<p class="box">Convergence criteria for eigenpairs in the search interval</p>
			</div>
		</div>

		{#if is_single_prec(params)}
		<div class="columns is-mobile"> <!-- fpm 7 -->
			<div class="column is-2">
				<div class="field">
					<div class="control">
						<input type=number class="input" bind:value="{fpm[7]}">
					</div>
				</div>
			</div>
			<div class="column is-small is-2">
				<p class="box is-family-code has-text-centered is-size-6">fpm(7) = {fpm[7]}</p>
			</div>
			<div class="column">
				<p class="box">Stopping convergence criteria for single precision (&epsilon = <var>10<sup>-<i>x</i></sup></var>)</p>
			</div>
		</div>
		{/if}

		{#if !is_hermitian(params)}
			<div class="columns is-mobile"> <!-- fpm 8 -->
				<div class="column is-2">
					{#if fpm[17]===1}
						<div class="field">
							<div class="control">
								<input type=number class="input" bind:value="{fpm[8]}" min=2 max=65536>
							</div>
						</div>
					{:else}
						<div class="field">
							<p class="control is-expanded">
								<span class="select is-fullwidth">
									<select bind:value="{fpm[2]}">
										{#each [2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,48,64,80,96,112] as points}
										<option value={points}>{points}</option>
										{/each}
									</select>
								</span>
							</p>
						</div>
					{/if}
				</div>
				<div class="column is-small is-2">
					<p class="box is-family-code has-text-centered is-size-6">fpm(8) = {fpm[8]}</p>
				</div>
				<div class="column">
					<p class="box"> Number of contour points (non-Hermitian only, full contour)</p>
				</div>
			</div>
		{/if}

		{#if !(params.storage_format === 'rci')}
		<div class="columns is-mobile"> <!-- fpm 10 -->
			<div class="column is-2">
				<div class="field">
					<p class="control is-expanded">
						<span class="select is-fullwidth">
							<select bind:value="{fpm[10]}">
								<option value={0}>No</option>
								<option value={1}>Yes</option>
							</select>
						</span>
					</p>
				</div>
			</div>
			<div class="column is-small is-2">
				<p class="box is-family-code has-text-centered is-size-6">fpm(10) = {fpm[10]}</p>
			</div>
			<div class="column">
				<p class="box">Store factorizations with the predefined interfaces</p>
			</div>
		</div>
		{/if}

		{#if is_hermitian(params)}
			<div class="columns is-mobile"> <!-- fpm 16 -->
				<div class="column is-2">
					<div class="field">
						<p class="control is-expanded">
							<span class="select is-fullwidth">
								<select bind:value="{fpm[16]}">
									<option value={0}>Gauss</option>
									<option value={1}>Trapezoidal</option>
									<option value={2}>Zolotarev</option>
								</select>
							</span>
						</p>
					</div>
				</div>
				<div class="column is-small is-2">
					<p class="box is-family-code has-text-centered is-size-6">fpm(16) = {fpm[16]}</p>
				</div>
				<div class="column">
					<p class="box"> Integration quadrature type (Hermitian only, half contour)</p>
				</div>
			</div>
		{/if}

		{#if !is_hermitian(params)}
			<div class="columns is-mobile"> <!-- fpm 17 -->
				<div class="column is-2">
					<div class="field">
						<p class="control is-expanded">
							<span class="select is-fullwidth">
								<select bind:value="{fpm[17]}">
									<option value={0}>Gauss</option>
									<option value={1}>Trapezoidal</option>
								</select>
							</span>
						</p>
					</div>
				</div>
				<div class="column is-small is-2">
					<p class="box is-family-code has-text-centered is-size-6">fpm(17) = {fpm[17]}</p>
				</div>
				<div class="column">
					<p class="box"> Integration quadrature type (non-Hermitian only, full contour)</p>
				</div>
			</div>
		{/if}

		<div class="columns">
			<div class="column control">
	  		<textarea class="textarea has-fixed-size is-family-code" rows={param_diff.count} readonly>{param_diff.text.trim()}</textarea>
			</div>
		</div>

	</div>

</main>

<footer class="footer">
	<div class="content has-text-centered">
    <p>
      <a href="https://github.com/spacedome/feast-config">github</a> &middot 2020
    </p>
  </div>
</footer>

<style global src="./scss-entrypoint.scss"></style>
