<script lang="ts">
    import Drawer from "$lib/Drawer.svelte";

    let showModal = true;
</script>

<button on:click={() => showModal = true}>Open</button>


{#if showModal}
    <Drawer on:close={() => showModal = false} />
{/if}