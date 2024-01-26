<script lang="ts">
    import { createEventDispatcher, onMount } from "svelte";

    /**
     * Warning. If you set the animation variable to "transition", it will override the css prefers-reduced-motion media feature.
     */
    export let animate: "transition" | "none" | "auto" = "auto";

    /**
     * Toggle this to true when you want to close the drawer from the outside.
     */
    export let closeDrawer = false;

    $: {
        if (closeDrawer) {
            close();
            closeDrawer = false;
        }
    }

    let autoAnimate = true;
    $: actuallAnimate = animate === "transition" ? true : animate === "auto" ? autoAnimate : false;

    let backgroundColor = "#00000000";
    let translateY = "100%";
    const dispatch = createEventDispatcher();

    function close() {
        backgroundColor = "#00000000";
        translateY = "100%";
        setTimeout(() => {
            dispatch("close");
        }, actuallAnimate ? 300 : 0);
    }

    onMount(() => {
        const isReduced = !(window.matchMedia(`(prefers-reduced-motion: reduce)`) as any as boolean === true || window.matchMedia(`(prefers-reduced-motion: reduce)`).matches === true);
        autoAnimate = isReduced;

        setTimeout(() => {
            backgroundColor = "#00000081";
            translateY = "0%"
        }, actuallAnimate ? 1 : 0);
    });
</script>

<section class="outer {actuallAnimate ? "animate" : ""}" style="--backgroundColor:{backgroundColor};--translateY:{translateY};">
    <button class="closeBackground" on:click={close} />
    <section class="inner">
        <div class="header">
            wow
            <button on:click={close}>close</button>
        </div>
        <div class="rest">
            <slot />
        </div>
    </section>
</section>

<style>
    .outer {
        background-color: var(--backgroundColor);
        position: fixed;
        width: 100vw;
        height: 100vh;
        height: 100dvh;
        top: 0;
        left: 0;
    }

    .outer.animate {
        transition: background-color 300ms linear;
    }

    @media (prefers-reduced-motion: reduce) {
        .outer.animate {
            transition: background-color 0 linear !important;
        }
    }

    .inner {
        position: absolute;
        left: 0;
        top: 40px;
        height: calc(100% - 40px);
        width: 100%;
        background-color: white;
        transform: translateY(var(--translateY));
        border-top-right-radius: 10px;
        border-top-left-radius: 10px;
    }

    .outer.animate .inner {
        transition: transform 300ms ease-out;
    }

    .closeBackground {
        width: 100%;
        height: 40px;
        border: none;
        background: none;
        outline: none;
    }
</style>