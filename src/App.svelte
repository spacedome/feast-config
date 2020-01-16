<script>
	export let name;
</script>

<main class="section">
	<div class="container">
		<h1 class="title">
		  {name}
		</h1>

		<p class="subtitle">
		  An interface for generating FEAST parameters
		</p>

		<div class="field">
		  <p class="control">
		    <span class="select">
		      <select>
		        <option>FEAST 4.0</option>
						<option>FEAST 3.0</option>
						<option>FEAST 2.0</option>
		      </select>
		    </span>
		  </p>
		</div>

		<div class="buttons">
		  <a class="button is-primary" href="/">Primary</a>
		  <a class="button is-link" href="/">Link</a>
		</div>
	</div>
</main>

<footer class="footer">
	<div class="content has-text-centered">
    <p>
      <a href="https://github.com/spacedome">github</a> &middot 2020
    </p>
  </div>
</footer>

<style global src="./scss-entrypoint.scss"></style>
