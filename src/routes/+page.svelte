<script lang="ts">
    import Drawer from "$lib/Drawer.svelte";

    let showModal = true;
    let closeModalForce = false;

    function closeModal() {
        closeModalForce = true;
        setTimeout(() => closeModalForce=false, 400);
    }
</script>

<button on:click={() => showModal = true}>Open</button>


{#if showModal}
    <Drawer on:close={() => showModal = false} closeDrawer={closeModalForce}>
        wow
        <button on:click={closeModal}>Close</button>
    </Drawer>
{/if}