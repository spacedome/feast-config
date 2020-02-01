<script>

	export let name;

	import InterfaceSelect from './InterfaceSelect.svelte';
	import InterfaceTable from './InterfaceTable.svelte';
	import FPMSelect from './FPMSelect.svelte';
	import FPMInteger from './FPMInteger.svelte';

	let params = {
		version:	4,
		symmetry: 's', // s: symmetric, h: hermitian, g: general
		prob_type: 'ev', // ev: sstandard, gv: generalized, pev: polynomial
		data_type: 'd', // s: single, d: double, c: comp single, z: comp double
		storage_format: 'dense', // dense, sparse, banded, rci
		expert: false, // false (default), true (use custom contor)
		algorithm: 'standard', // standard, inexact, or parallel FEAST, or both
	}

	// symm is dependant on the data type - cannot be real hermitian only real symmetic
	$: params.symmetry = (params.symmetry === 'h' && params.data_type === 'd') ? 's' : params.symmetry;
	// there is no banded polynomial solver
	$: params.prob_type = (params.prob_type === 'pev' && params.storage_format === 'banded') ? 'ev' : params.prob_type;
	// IFEAST and PFEAST only for sparse
	$: params.algorithm = (params.storage_format !== 'sparse') ? 'standard' : params.algorithm;

	const fpm_init = { //given values are feastinit defaults
		1: 0,  // Print runtime comments on screen, 0: no, 1: yes
		2: 8,  // number contour points for Hermitian (half-contour), if fpm(16=0,2) can be {1 to 20, 24, 32, 40, 48, 56}, if fpm(16)=1 all values permitted
		3: 12, // stopping convergence criteria for double prec (10^(-x))
		4: 20, // max number of FEAST loops
		5: 0,  // provide inital guess subspace, 0: no, 1: yes
		6: 1,  // conv critera for eigpair in search interval, 0: rel err on trace epsout, 1: rel residual max(res)
		7: 5,  // stopping convergence criteria for single prec (10^(-x))
		8: 16, // number contour points for non-Hermitian (full-contour), if fpm(17)=0 can be {2 to 40, 48, 64, 80, 96, 112}, if fpm(17)=1 all values permitted
		// 9: "MPI_COMM_WORLD", // user defined MPI communicator UNDOCUMENTED
		10: 1, // store factorizations with predefined interfaces 0: no, 1 : yes
		16: 0, // hermitian integration type, 0: gauss, 1:trapezoid, 2: zolotarev
		14: 0, // estimate, 0: normal feast, 1: return subspace Q after est, 2: stochastic estimate
		15: 0, // NH contour scheme, 0: two sided, 1: one sided, 2: one sided complex symmetric
		// 17: 1, // non-hermitian integration type, 0: gauss, 1:trapezoid DEPRECIATED
		18: 30, // contour ratio (x >= 0) ACTUAL DEFAULT IS 100 - but inital state should be 30 !!!
		19: 0, // contour rotation angle (-180 <= x <= 180)
		40: 0, // search interval options, 0: normal feast, 1: largest eigvals, 2: smallest eigvals
		41: 1, // scale matrix, 0: no, 1: yes
		42: 1, // mixed precision, 0: full prec, 1: mixed
		43: 0, // switch sparse to sparse_iterative, 0: feast, 1: ifeast (sparse only)
		// 44: 0, // type of iterative solver, 0: bicgstab
		45: 1, // accuracy of iterative solver, 10^-x
		46: 40, // max inner iterations, ifeast only
	}

	let fpm = JSON.parse(JSON.stringify(fpm_init)); // deep clone

	// // just to "trick" svelte compiler into not making variables depend on themselves
	// function set_fpm(x, v) {
	// 	fpm[x] = v;
	// }

	// $: if (params.algorithm === 'inexact' || params.algorithm === 'both') { set_fpm(4, 50) } else { set_fpm(4, 20) };

	function fpm_default(parameters, fparam) {
		let defaults = JSON.parse(JSON.stringify(fpm_init));
		if (fparam[14] === 2) {
			defaults[2] = 3;
		}
		if (parameters.algorithm === 'inexact' || parameters.algorithm === 'both') {
			defaults[4] = 50;
			defaults[16] = 1;
			defaults[2] = 4;
			defaults[8] = 8;
		}
		if (parameters.symmetry === 'g') {
			defaults[16] = 1;
		}
		if (parameters.data_type === 'z' && parameters.symmetry === 's') {
			defaults[15] = 2;
			defaults[16] = 1;
		}
		if (fparam[14] == 2) {
			defaults[8] = 6;
			defaults[15] = 1;
		}
		if ((parameters.algorithm !== 'inexact' && parameters.algorithm !== 'both' && parameters.prob_type !== 'pev') && ((parameters.symmetry === 's' && parameters.data_type === 'd' ) || parameters.symmetry === 'h')) {
			defaults[18] = 30;
		} else {
			defaults[18] = 100;
		}
		return defaults;
	}

	function fpm_set_default_fpm(p) {
		switch (p) {
			case 2:
				if (is_ifeast()) {
					fpm[2] = 4;
				} else if (fpm[14]===2) {
					fpm[2] = 3;
				} else {
					fpm[2] = 8;
				}
				break;
			case 4:
				fpm[4] =  is_ifeast() ? 50 : 20;
				break;
			case 8:
				if (fpm[14]===2) {
					fpm[8] = 6;
				} else if (is_ifeast()) {
					fpm[8] = 8;
				} else {
					fpm[8] = 16;
				}
				break;
			case 15:
				if (fpm[14]===2) {
					fpm[15] = 1;
				} else if (is_complexsym()) {
					fpm[15] = 2;
				} else {
					fpm[15] = 0;
				}
				break;
			case 16:
				fpm[16] = (is_ifeast() || is_complexsym()) ? 1 : 0;
				break;
			case 18:
				fpm[18] = ((!is_ifeast() && params.prob_type !== 'pev') && (is_hermitian() || (params.data_type === d && params.symmetry === 's'))) ? 30 : 100;
				break;
			default:
				break;
		}
	}

	function is_ifeast() {
		return (params.algorithm === 'inexact' || params.algorithm === 'both');
	}

	function is_complexsym() {
		return (params.data_type === 'z' && params.symmetry === 's');
	}

	function is_hermitian() {
		return (params.symmetry === 's' || params.symmetry === 'h');
	}

	function is_real(parameters) {
		if (parameters.prob_type === 'pev') {
			return false
		} else {
			return (parameters.data_type === 'd' && parameters.symmetry === 's') || (parameters.data_type === 'z' && parameters.symmetry === 'h');
		}
	}

	function is_single_prec(parameters) {
		return (parameters.data_type === 's' || parameters.data_type === 'c');
	}

	function is_sparse(parameters) {
		return (parameters.storage_format === 'sparse')
	}

	function fpm_set_default_param() {
		fpm = fpm_default(params, fpm);
	}

	function feast_call(parameters) {
		let T = 'd';
		let Y = 's';
		let F = 'f';
		let P = 'ev';
		let PFEAST = (parameters.algorithm === 'parallel' || parameters.algorithm === 'both' ? 'p' : '');
		let IFEAST = (parameters.algorithm === 'inexact' || parameters.algorithm === 'both' ? 'i' : '');
		let X = (parameters.expert ? 'x' : '' );
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
		switch (parameters.symmetry) {
			case 's': // symmetric
			case 'h': // hermitian
			case 'g': // general
				Y = parameters.symmetry;
				break;
			default:
				throw 'Error';
		}
		switch (parameters.prob_type) {
			case 'ev': // symmetric
			case 'gv': // hermitian
			case 'pev': // general
				P = parameters.prob_type;
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
				P = '';
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
		return PFEAST + T + IFEAST + "feast_" + Y + F + P + X;
	}

	function feast_arguments(parameters) {
		let list = "{List}";
		let listi = "{List-I}";
		let X = (parameters.expert ? ",Zne,Wne" : "");
		listi = (!is_real(parameters) || parameters.prob_type === 'pev')  ? "Emid,r" : "Emin,Emax";
		if (parameters.storage_format === 'rci') {
			if (parameters.prob_type === 'pev') {
				list = "ijob,dmax,N,Ze,zwork1,zwork2,zAq,zBq";
			} else if (parameters.data_type === 'd' && parameters.symmetry === 's') {
				list = "ijob,N,Ze,work1,zwork2,Aq,Bq";
			} else {
				list = "ijob,N,Ze,zwork1,zwork2,zAq,zBq";
			}
		} else if (parameters.storage_format === 'dense') {
			list = "N,A,LDA";
			if (parameters.prob_type === 'pev') {
				list = "dmax," + list;
			}
			if (!(parameters.symmetry === 'g')) {
				list = "UPLO," + list;
			}
			list += (parameters.prob_type === 'gv') ? ",B,LDB" : "";
		} else if (parameters.storage_format === 'banded') {
			if (parameters.symmetry === 'g') {
				list = "N,kla,kua,A,LDA";
				list += (parameters.prob_type === 'gv') ? ",klb,kub,B,LDB" : "";
			} else {
				list = "UPLO,N,kla,A,LDA";
				list += (parameters.prob_type === 'gv') ? ",klb,B,LDB" : "";
			}
		} else if (parameters.storage_format === 'sparse') {
			list = "N,A,IA,JA";
			if (parameters.prob_type === 'pev') {
				list = "dmax," + list;
			}
			if (!(parameters.symmetry === 'g')) {
				list = "UPLO," + list;
			}
			list += (parameters.prob_type === 'gv') ? ",B,IB,JB" : "";
		}

		return list + ",fpm,epsout,loop," + listi + ",M0,E,X,M,res,info" + X;
	}

	function input_params(feastparams, feastparams_default) {
		let fortran = "integer, dimension(64) :: fpm\ncall feastinit(fpm)\n";
		let count = 2;
		for (let key in feastparams) {
		  if (feastparams[key] !== feastparams_default[key]) {
				fortran += "fpm(" + key + ") = " + feastparams[key] + "\n";
				count +=1;
			}
		}
		return {count: count, text: fortran};
	}

	$: fpm_computed_defaults = fpm_default(params, fpm);
	$: param_diff = input_params(fpm, fpm_computed_defaults);

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
					Currently only supports version 4.0.
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


		<!-- version -->
		<!-- <InterfaceSelect bind:value={params.version}
		 description="FEAST version. Parameters are forward compatible. v2.1 is used in MKL">
			<option value={4}>FEAST 4.0</option>
			<option value={2}>FEAST 2.1</option>
		</InterfaceSelect> -->

		 <!-- data type -->
		<InterfaceSelect bind:value={params.data_type}
		 description="Matrix data type">
			<option value="{'d'}">Double Precision (d)</option>
			<!-- <option value="{'s'}">Single Precision (s)</option> -->
			<option value="{'z'}">Complex Double (z)</option>
			<!-- <option value="{'c'}">Complex Single (c)</option> -->
		</InterfaceSelect>

		<!-- storage format -->
		<InterfaceSelect bind:value={params.storage_format}
		 on:change="{() => fpm_set_default_param()}"
		 description="Matrix storage format or Reverse Communication Interface (RCI)">
			<option value="{'dense'}">Dense</option>
			<option value="{'banded'}">Banded</option>
			<option value="{'sparse'}">Sparse</option>
			<option value="{'rci'}">RCI</option>
		</InterfaceSelect>

		<!-- problem symmetry -->
		<InterfaceSelect bind:value={params.symmetry}
		 on:change="{() => fpm_set_default_param()}"
		 description="Symmetry of problem">
			<option value="{'s'}">Symmetric (s)</option>
			<option value="{'h'}" disabled={params.data_type === 'd' || null}>Hermitian (h)</option>
			<option value="{'g'}">General (g)</option>
		</InterfaceSelect>


		<!-- problem type -->
		<InterfaceSelect bind:value={params.prob_type}
		 on:change="{() => fpm_set_default_param()}"
		 description="Form of eigenvalue problem">
			<option value="{'ev'}">Standard (ev)</option>
			<option value="{'gv'}">Generalized (gv)</option>
			<option value="{'pev'}" disabled={params.storage_format === 'banded' || null}>Polynomial (pev)</option>
		</InterfaceSelect>

		<!-- algorithm type -->
		<InterfaceSelect bind:value={params.algorithm}
		 disabled="{params.storage_format !== 'sparse' || null}"
		 on:change="{() => fpm_set_default_param()}"
		 description="FEAST algorithm extension, inexact (IFEAST), or parallel (PFEAST) - sparse only">
			<option value="{'standard'}">FEAST</option>
			<option value="{'inexact'}" disabled={params.storage_format !== 'sparse' || null}>IFEAST</option>
			<option value="{'parallel'}" disabled={params.storage_format !== 'sparse' || null}>PFEAST</option>
			<option value="{'both'}" disabled={params.storage_format !== 'sparse' || null}>PIFEAST</option>
		</InterfaceSelect>

		<!-- version -->
		<InterfaceSelect bind:value={params.expert}
		 description="Use expert routine (for manually specifying a custom contour)">
			<option value={false}>No</option>
			<option value={true}>Yes</option>
		</InterfaceSelect>

	</div>

	<div class="container">

		<h2 class="title is-4">
			Runtime Options (<span class="is-family-code">fpm</span>)
		</h2>

		<!-- fpm 1 -->
		<FPMSelect bind:value={fpm[1]} fpmIndex={1}
		 description="Print runtime comments on screen">
			<option value={0}>No</option>
			<option value={1}>Yes</option>
		</FPMSelect>


		<!-- fpm 3  -->
		<FPMInteger bind:value={fpm[3]} fpmIndex={3} min=0 max=16
		 description="Stopping convergence criteria for double precision (&epsilon = <var>10<sup>-<i>x</i></sup></var>)"/>

		<!-- fpm 4 -->
		<FPMInteger bind:value={fpm[4]} fpmIndex={4} min={0}
		 description="Maximum number of FEAST refinement iterations"/>

		<!-- fpm 5 -->
		<FPMSelect bind:value={fpm[5]} fpmIndex={5}
		 description="Provide initial guess subspace">
			<option value={0}>No</option>
			<option value={1}>Yes</option>
		</FPMSelect>

		<!-- fpm 6 -->
		<FPMSelect bind:value={fpm[6]} fpmIndex={6}
		 description="Convergence criteria for eigenpairs in the search interval">
			<option value={1}>Relative Residual</option>
			<option value={0}>Relative Trace</option>
		</FPMSelect>

		<!-- fpm 7 DEPRECIATED IN 4.0 -->
		<!-- {#if is_single_prec(params)}
		<FPMInteger bind:value={fpm[7]} fpmIndex={7}
		 description="Stopping convergence criteria for single precision (&epsilon = <var>10<sup>-<i>x</i></sup></var>) - depreciated in 4.0"/>
		{/if} -->

		<!-- fpm 9 -->
		<!-- <FPMSelect bind:value={fpm[9]} fpmIndex={9} nonum={true}
		description="Used to set user defined MPI communicator">
			<option value={'MPI_COMM_WORLD'}>MPI_COMM_WORLD</option>
			<option value={'USER_DEFINED'}>USER_DEFINED</option>
		</FPMSelect> -->

		<!-- fpm 10 -->
		<FPMSelect bind:value={fpm[10]} fpmIndex={10}
		disabled="{params.storage_format === 'rci' || null}"
		description="Store factorizations with the predefined interfaces">
			<option value={0}>No</option>
			<option value={1}>Yes</option>
		</FPMSelect>

		<!-- fpm 14 -->
		<FPMSelect bind:value={fpm[14]} fpmIndex={14}
		 on:change="{() => {fpm_set_default_fpm(2); fpm_set_default_fpm(8); fpm_set_default_fpm(15)}}"
		 description="Modes: Standard, subspace Q after 1 contour, or stochastic estimate eigval count">
			<option value={0}>Standard</option>
			<option value={1}>Subspace</option>
			<option value={2}>Stochastic</option>
		</FPMSelect>

		<h2 class="title is-4">
			Integration Options (<span class="is-family-code">fpm</span>)
		</h2>

		<!-- fpm 16 -->
		<FPMSelect bind:value={fpm[16]} fpmIndex={16}
		description="Integration quadrature type (Zolotarev for Hermitian only)">
			<option value={0}>Gauss</option>
			<option value={1}>Trapezoidal</option>
			<option value={2} disabled={!is_real(params) || null}>Zolotarev</option>
		</FPMSelect>

		<!-- fpm 17 DEPRECIATED IN 4.0 -->
		<!-- {#if !is_hermitian()}
			<FPMSelect bind:value={fpm[17]} fpmIndex={17}
			description="Integration quadrature type (non-Hermitian only, full contour) DEPRECIATED IN 4.0">
				<option value={0}>Gauss</option>
				<option value={1}>Trapezoidal</option>
			</FPMSelect>
		{/if} -->

		<!-- fpm 2 -->
		{#if is_hermitian()}
			{#if fpm[16]===1}
				<FPMInteger bind:value={fpm[2]} fpmIndex={2} min=1
					description="Number of contour points (Hermitian only, half contour)"/>
			{:else}
				<FPMSelect bind:value={fpm[2]} fpmIndex={2}
					description="Number of contour points (Hermitian only, half contour)">
					{#each [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,24,32,40,48,56] as points}
					<option value={points}>{points}</option>
					{/each}
				</FPMSelect>
			{/if}
		{/if}

		<!-- fpm 8 -->
		{#if !is_hermitian()}
			{#if fpm[16]===1}
				<FPMInteger bind:value={fpm[8]} fpmIndex={8} min=2
					description="Number of contour points (non-Hermitian only, full contour)"/>
			{:else}
				<FPMSelect bind:value={fpm[8]} fpmIndex={8}
					description="Number of contour points (non-Hermitian only, full contour)">
					{#each [2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,48,64,80,96,112] as points}
					<option value={points}>{points}</option>
					{/each}
				</FPMSelect>
			{/if}
		{/if}

		<!-- fpm 15 -->
		{#if !is_hermitian()}
				<FPMSelect bind:value={fpm[15]} fpmIndex={15}
					description="Contour scheme for non Hermitian problem (CS - complex symmetric)">
					<option value={0}>Two-sided</option>
					<option value={1}>One-sided</option>
					<option value={0}>One-sided CS</option>
				</FPMSelect>
		{/if}

		<!-- fpm 18 -->
		<FPMInteger bind:value={fpm[18]} fpmIndex={18} min=1 max=100
			description="Contour elliptical ratio (scale vertically by x/100)"/>

		<!-- fpm 19 -->
		{#if !is_real(params)}
			<FPMInteger bind:value={fpm[19]} fpmIndex={19} min=-180 max=180
				description="Rotate elliptical contour by x degrees (-180 <= x <= 180)"/>
		{/if}

		<h2 class="title is-4">
			Driver Options (<span class="is-family-code">fpm</span>)
		</h2>
		<!-- fpm 40 -->
		<FPMSelect bind:value={fpm[40]} fpmIndex={40}
		disabled="{!is_real(params) || null}"
		description="Search interval options, set to search for largest or smallest (real) eigvals">
			<option value={0}>Normal</option>
			<option value={1}>Largest</option>
			<option value={2}>Smallest</option>
		</FPMSelect>

		<!-- fpm 41 -->
		<FPMSelect bind:value={fpm[41]} fpmIndex={41}
		disabled="{!is_sparse(params) || null}"
		description="Scale matrix (sparse only)">
			<option value={0}>No</option>
			<option value={1}>Yes</option>
		</FPMSelect>

		<!-- fpm 42 -->
		<FPMSelect bind:value={fpm[42]} fpmIndex={42}
		description="Mixed or full precision (internally)">
			<option value={0}>Full</option>
			<option value={1}>Mixed</option>
		</FPMSelect>

		<!-- fpm 43 -->
		<FPMSelect bind:value={fpm[43]} fpmIndex={43}
		disabled="{!is_sparse(params) || null}"
		description="Set to use IFEAST solver with old FEAST interface (sparse only)">
			<option value={0}>FEAST</option>
			<option value={1}>IFEAST</option>
		</FPMSelect>

		<!-- fpm 45 -->
		<FPMInteger bind:value={fpm[45]} fpmIndex={45} min=0 max=10 disabled="{!is_ifeast() || null}"
			description="Accuracy of iterative solver 10^-x (IFEAST only)"/>

		<!-- fpm 46 -->
		<FPMInteger bind:value={fpm[46]} fpmIndex={46} min=1 disabled="{!is_ifeast() || null}"
			description="Maximum iterations of inner iterative system solver (IFEAST only)"/>

		<div class="columns">
			<div class="column control">
	  		<textarea class="textarea has-fixed-size is-family-code" rows={param_diff.count+1} readonly>{param_diff.text.trim()}
{feast_call(params)}({feast_arguments(params)})</textarea>
			</div>
		</div>

		<InterfaceTable dataType={params.data_type} real={is_real(params)} probType={params.prob_type} expert={params.expert}/>

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
