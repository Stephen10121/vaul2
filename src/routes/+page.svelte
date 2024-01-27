<script lang="ts">
    import Drawer from "$lib/Drawer.svelte";

    let showModal = false;
    let closeModalForce = false;

    function closeModal() {
        closeModalForce = true;
        setTimeout(() => closeModalForce=false, 400);
    }
</script>

<main>
    <button on:click={() => showModal = true}>Open</button>
</main>

{#if showModal}
    <Drawer on:close={() => showModal = false} closeDrawer={closeModalForce} showHeaderBar>
    </Drawer>
{/if}

<style>
    main {
        width: 100%;
        height: 100vh;
        height: 100dvh;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    button {
        padding: 10px 17px;
        font-family: 'Open Sans', sans-serif;
        font-size: 0.75rem;
        font-weight: bold;
        border-radius: 100vw;
        border: 1px solid #adadad;
        background: none;
        cursor: pointer;
        transition: background 0.15s linear;
    }

    button:hover {
        background-color: #f7f7f7;
    }
</style>