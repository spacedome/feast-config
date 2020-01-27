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

	const fpm_init = { //given values are feastinit defaults
		1: 0,  // Print runtime comments on screen, 0: no, 1: yes
		2: 8,  // number contour points for Hermitian (half-contour), if fpm(16=0,2) can be {1 to 20, 24, 32, 40, 48, 56}, if fpm(16)=1 all values permitted
		3: 12, // stopping convergence criteria for double prec (10^(-x))
		4: 20, // max number of FEAST loops
		5: 0,  // provide inital guess subspace, 0: no, 1: yes
		6: 1,  // conv critera for eigpair in search interval, 0: rel err on trace epsout, 1: rel residual max(res)
		7: 5,  // stopping convergence criteria for single prec (10^(-x))
		8: 16, // number contour points for non-Hermitian (full-contour), if fpm(17)=0 can be {2 to 40, 48, 64, 80, 96, 112}, if fpm(17)=1 all values permitted
		9: "MPI_COMM_WORLD", // user defined MPI communicator
		10: 1, // store factorizations with predefined interfaces 0: no, 1 : yes
		16: 0, // hermitian integration type, 0: gauss, 1:trapezoid, 2: zolotarev
		17: 1, // non-hermitian integration type, 0: gauss, 1:trapezoid
	}

	let fpm = JSON.parse(JSON.stringify(fpm_init)); // deep clone
	let fpm_default = JSON.parse(JSON.stringify(fpm_init)); // deep clone

	function is_hermitian(parameters) {
		return (parameters.symmetry === 's' || parameters.symmetry === 'h');
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
		  if (!(feastparams[key] === feastparams_default[key])) {
				fortran += "fpm(" + key + ") = " + feastparams[key] + "\n";
				count +=1;
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

		<!-- problem symmetry -->
		<InterfaceSelect bind:value={params.symmetry}
		 description="Symmetry of problem">
			<option value="{'s'}">Symmetric (s)</option>
			<option value="{'h'}" disabled={params.data_type === 'd' || null}>Hermitian (h)</option>
			<option value="{'g'}">General (g)</option>
		</InterfaceSelect>

		<!-- problem type -->
		<InterfaceSelect bind:value={params.prob_type}
		 description="Form of eigenvalue problem">
			<option value="{'ev'}">Standard (ev)</option>
			<option value="{'gv'}">Generalized (gv)</option>
			<option value="{'pev'}" disabled={params.storage_format === 'banded' || null}>Polynomial (pev)</option>
		</InterfaceSelect>

		<!-- storage format -->
		<InterfaceSelect bind:value={params.storage_format}
		 description="Matrix storage format or Reverse Communication Interface (RCI)">
			<option value="{'dense'}">Dense</option>
			<option value="{'banded'}">Banded</option>
			<option value="{'sparse'}">Sparse</option>
			<option value="{'rci'}">RCI</option>
		</InterfaceSelect>

		<!-- algorithm type -->
		<InterfaceSelect bind:value={params.algorithm}
		 disabled={!(params.storage_format === 'sparse') || null}
		 description="FEAST algorithm extension, inexact (IFEAST), or parallel (PFEAST) - sparse only">
			<option value="{'standard'}">FEAST</option>
			<option value="{'inexact'}">IFEAST</option>
			<option value="{'parallel'}">PFEAST</option>
			<option value="{'both'}">P/IFEAST</option>
		</InterfaceSelect>

		<!-- version -->
		<InterfaceSelect bind:value={params.expert}
		 description="Use expert routine (for manually specifying a custom contour)">
			<option value={false}>No</option>
			<option value={true}>Yes</option>
		</InterfaceSelect>

		<div class="columns">
			<div class="column control">
	  		<textarea class="textarea has-fixed-size is-family-code" rows="1" readonly>{feast_call(params)}({feast_arguments(params)})</textarea>
			</div>
		</div>

		<InterfaceTable dataType={params.data_type} real={is_real(params)} probType={params.prob_type} expert={params.expert}/>

	</div>

	<div class="container">

		<h2 class="title is-4">
			FEAST Input Parameters (<span class="is-family-code">fpm</span>)
		</h2>

		<!-- fpm 1 -->
		<FPMSelect bind:value={fpm[1]} fpmIndex={1} 
		 description="Print runtime comments on screen">
			<option value={0}>No</option>
			<option value={1}>Yes</option>		
		</FPMSelect>


		<!-- fpm 2 -->
		{#if is_hermitian(params)}
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

		<!-- fpm 3  -->
		{#if !is_single_prec(params)}
			<FPMInteger bind:value={fpm[3]} fpmIndex={3} min=0 max=16
		 description="Stopping convergence criteria for double precision (&epsilon = <var>10<sup>-<i>x</i></sup></var>)"/>
		{/if}

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
		{#if is_single_prec(params)}
		<FPMInteger bind:value={fpm[7]} fpmIndex={7}
		 description="Stopping convergence criteria for single precision (&epsilon = <var>10<sup>-<i>x</i></sup></var>) - depreciated in 4.0"/>
		{/if}

		<!-- fpm 8 -->
		
		{#if !is_hermitian(params)}
			{#if fpm[17]===1}
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

		<!-- fpm 9 -->
		<FPMSelect bind:value={fpm[9]} fpmIndex={9} nonum={true}
		description="Used to set user defined MPI communicator">
			<option value={'MPI_COMM_WORLD'}>MPI_COMM_WORLD</option>
			<option value={'USER_DEFINED'}>USER_DEFINED</option>
		</FPMSelect>

		<!-- fpm 10 -->
		<FPMSelect bind:value={fpm[10]} fpmIndex={10} 
		disabled={params.storage_format === 'rci' || null}
		description="Store factorizations with the predefined interfaces">
			<option value={0}>No</option>
			<option value={1}>Yes</option>
		</FPMSelect>

		<!-- fpm 16 -->
		<FPMSelect bind:value={fpm[16]} fpmIndex={16} 
		description="Integration quadrature type (Zolotarev for Hermitian only)">
			<option value={0}>Gauss</option>
			<option value={1}>Trapezoidal</option>
			<option value={2} disabled={!is_real(params) || null}>Zolotarev</option>
		</FPMSelect>

		<!-- fpm 17 DEPRECIATED IN 4.0 -->
		<!-- {#if !is_hermitian(params)}
			<FPMSelect bind:value={fpm[17]} fpmIndex={17} 
			description="Integration quadrature type (non-Hermitian only, full contour) DEPRECIATED IN 4.0">
				<option value={0}>Gauss</option>
				<option value={1}>Trapezoidal</option>
			</FPMSelect>
		{/if} -->

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
