<template>
	<div id="layout">
		<label for="userbox">Letterboxd username:</label>
		<input
			class="userfield"
			placeholder="ex: holopollock"
			id="userbox"
			type="text"
			v-on:keyup.enter="submit()"
			v-model="users"
		/>
		<button v-on:click="submit()">Submit</button>
		<div v-if="started" class="output">
			<p id="loadbar" v-bind:class="{ loaded: done }">Loading...</p>
		</div>
		<div v-if="done" class="output">
			<p><strong>{{ women }}</strong> films directed by women</p>
			<p>of total films {{ total }}</p>
			<p>Percentage of films directed by women: <strong>{{ percentage }}</strong></p>
		</div>
		<div v-if="done" class="output" id="footer">
			<p>This site is a project of <a href="http://goodbyte.ca">Goodbyte</a> &copy;2020</p>
			<p>Data provided by <a href="https://letterboxd.com">Letterboxd</a> and <a href="https://themoviedb.org">TMDB</a></p>
		</div>
	</div>
</template>

<style scoped>
#layout
{
	padding: 40px;
	max-width: 500px;
	margin: 100px auto;
}

label
{
	display: block;
	margin: 10px 0;
}

input
{
	width: calc(100% - 105px);
	
	font-family: "IBM Plex Mono", monospace;
	font-size: 1rem;
	line-height: 2;
	color: inherit;
	border: none;
	border-bottom: 1px currentColor solid;
	outline: none;
}

button
{
	cursor: pointer;
	padding: 0 1ch;
	margin-left: 20px;
	
	font-family: "IBM Plex Mono", monospace;
	font-size: 1rem;
	line-height: 2;
	color: inherit;
	background: transparent;
	border: 1px solid currentColor;
	outline: none;
}

button:hover, button:focus-visible
{
	background: #eaeaea;
}

a
{
	text-decoration: none;
	color: inherit;
	font-weight: bold;
	outline: none;
}

a:hover, a:focus-visible
{
	text-decoration: underline;
}

.output
{
	margin-top: 60px;
}

#loadbar::after
{
	content: ".";
	color: transparent;
	display: inline-block;
	width: calc(100% - 10ch); /* 10ch ~= width of "Loading..." text */
	
	/* creates a ... effect that takes up the full width */
	background-image: radial-gradient(black 10%, transparent 20%);
	background-size: 1ch 1ch;
	background-repeat: space no-repeat;
	background-position: bottom right;
	
	animation: loading 45s steps(50);
	animation-fill-mode: forwards;
}

#loadbar.loaded::after
{
	animation-play-state: paused;
}

#footer
{
	animation: delay-reveal 1s steps(1);
	opacity: 1;
}

@keyframes delay-reveal
{
	from { opacity: 0; }
}

@keyframes loading
{
	from { width: 0px; }
}
</style>

<script>
export default {
	name: "Film",
	data() {
		return {
			users: "",
			info: "",
			started: false,
			done: false
		};
	},
	methods: {
		submit() {
			this.started = true;
			this.done = false;
			let inputted = this.users.split(/(?:,| )+/); //split input field on space or comma
			let userlist = inputted.filter(function(el) {
				return el;
			});
			let url = "/api?users=" + userlist.join("&users=");
			try {
				let vue = this;
				fetch(url)
					.then(function(res) {
						if (res.status != 200) {
							return "";
						}
						return res.json();
					})
					.then(function(json) {
						vue.info = json;
						let tmdbPersonURL = 'https://www.themoviedb.org/person/'
						console.log(tmdbPersonURL + vue.info.notFound.join('\n' + tmdbPersonURL))
						vue.done = true;
					});
			} catch (e) {
				alert("Something went wrong. Please try again in a moment. Error:" + e, "An error occured");
			}
		}
	},
	computed: {
		women: function() {
			return this.info.women;
		},
		total: function() {
			return this.info.total;
		},
		percentage: function() {
			return this.info.percentage.toFixed(3);
		}
	}
};
</script>
