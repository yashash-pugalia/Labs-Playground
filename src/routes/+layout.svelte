<script>
	import 'iconify-icon';
	import '../app.css';

	let { children } = $props();
</script>

<nav class="p-4 flex gap-2 items-center mx-auto max-w-3xl">
	<h1 class="text-2xl font-bold">
		Labs Playground by <a href="https://yashash.dev">yashash.dev</a>
	</h1>

	<div class="mr-auto"></div>

	<a href="/" class="px-4 py-2 bg-blue-100 rounded">Home</a>
	<a href="/tax" class="px-4 py-2 bg-blue-100 rounded">Tax</a>
</nav>

{@render children()}
