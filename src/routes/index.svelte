<script>
  import '$lib/css/smui.css';
  const host = "http://localhost:10012"
  const servers = fetch(`${host}/?{"method":"endpoints"}`).then(d => d.json())
</script>

<h1> Bookworms on this server </h1>

{#await servers}
  Fetching server list...
{:then server_list} 
<ul>
  {#each server_list.data as server}
    <li>
      <a href="{server}">{server}</a>
    </li>
  {/each}  
</ul>
{/await}