<script>
  import '$lib/css/smui.css';
  import BookBrowser from '$lib/Bookworm.svelte';
  import { page } from '$app/stores';
  const database = $page.params.bookworm;
  console.log({page})
  const host = `http://localhost:10012`;
</script>

<BookBrowser {host} {database}> </BookBrowser>