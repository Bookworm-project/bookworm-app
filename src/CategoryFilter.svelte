

<script>
    //import Multiselect from './Multiselect.svelte'

    export let array;
    export let field;
    let val = "";

    //  Use undefined instead of empty list to keep the syntax smaller.
    $: array = val == "" ? undefined : val.split(",") 
    $: console.log(array)
</script>


{field}: <input bind:value={val} />