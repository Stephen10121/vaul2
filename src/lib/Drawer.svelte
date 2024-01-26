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

    let top=40;
    function moveMouse(event: MouseEvent) {
        if (!expand) return;
        top = Math.max(event.pageY-oldPosOffset, 20);
    }
    let oldPosOffset = 0;
    function mouseDown(event:MouseEvent) {
        expand = true;
        oldPosOffset = event.offsetY;
    }
    let transitionTop = false;
    function mouseUp() {
        expand = false;

        if (top > 100) {
            close();
        } else {
            transitionTop=true;
            top = 40;
            setTimeout(() => transitionTop=false, 150);
            console.log(translateY)
        }
    }

    onMount(() => {
        const isReduced = !(window.matchMedia(`(prefers-reduced-motion: reduce)`) as any as boolean === true || window.matchMedia(`(prefers-reduced-motion: reduce)`).matches === true);
        autoAnimate = isReduced;

        setTimeout(() => {
            backgroundColor = "#00000081";
            translateY = "0%"
        }, actuallAnimate ? 1 : 0);
    });

    let expand = false;
</script>

<section class="outer {actuallAnimate ? "animate" : ""}" style="--backgroundColor:{backgroundColor};--translateY:{translateY};--topPos:{top}px;">
    <button class="closeBackground" on:click={close} />
    <section class="inner {transitionTop && "transitionTop"}">
        <div class="header">
            <div class="slider {expand && "expand"}" on:mousedown={mouseDown} on:mouseup={mouseUp} on:mousemove={moveMouse} on:mouseleave={mouseUp} role="none" />
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
        top: var(--topPos);
        height: calc(100% - 40px);
        width: 100%;
        background-color: white;
        transform: translateY(var(--translateY));
        border-top-right-radius: 10px;
        border-top-left-radius: 10px;
    }

    .outer.animate .inner.transitionTop {
        transition: top 0.15s linear;
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

    .header {
        width: 100%;
        height: 40px;
        position: relative;
    }

    .slider {
        background-color: rgba(255, 0, 0, 0.349);
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
    }

    .slider.expand {
        /* transform: translateY(-10px); */
        transform: translateY(-600px);
        height: 3000%;
    }
</style>