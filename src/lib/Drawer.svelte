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

    /**
     * This value plus the the default top of the drawer position will determine when the drawer will close when the user drags the drawer beyond this value.
     */
    export let closeHeight = 200;

    /**
     * This value determines how far away the top of the drawer is from the top of the page.
     */
    export let fromTop = 40;

    /**
     * This value set the maximum the user can drag the drawer upwards.
     */
    export let userMinDragHeight = 20;

    /**
     * Sets the maximum opacity of the background when the drawer is moving. Max value: 129, Min value: 0.
     */
    export let maximumDark = 129;

    /**
     * Sets the minimim opacity of the background when the drawer is moving. Max value: 129, Min value: 0.
     */
    export let minimumDark = 20;

    /**
     * Set this to true if you want a close button in the header section.
     */
    export let showCloseButton = false;

    /**
     * If you want to add a title to the drawer header, set this to the title.
     */
    export let headerTitle: string | undefined = undefined;

    /**
     * Set a font for the header title.
     */
    export let headerFont: string | undefined = undefined;

    /**
     * This shows a bar in the header section indicating that the drawer is draggable.
     */
    export let showHeaderBar = false;

    /**
     * If you dont what this drawer to be draggable, set this to false.
     */
    export let isDraggable = true;

    const dispatch = createEventDispatcher();
    let backgroundColor = "#00000000";
    let transitionTop = false;
    let translateY = "100%";
    let autoAnimate = true;
    let oldPosOffset = 0;
    let expand = false;
    let top = fromTop;

    function map_range(value: number, low1: number, high1: number, low2: number, high2: number) {
        return Math.floor(low2 + (high2 - low2) * (value - low1) / (high1 - low1)).toString(16);
    }

    $: actuallAnimate = animate === "transition" ? true : animate === "auto" ? autoAnimate : false;

    function close() {
        backgroundColor = "#00000000";
        translateY = "100%";
        setTimeout(() => {
            dispatch("close");
        }, actuallAnimate ? 300 : 0);
    }

    $: {
        if (closeDrawer) {
            close();
            closeDrawer = false;
        }
    }

    $: {
        if (top !== fromTop) {
            let minC = fromTop - userMinDragHeight;
            let maxC = window.innerHeight;
            backgroundColor = `#000000${map_range(top, minC, maxC, maximumDark, minimumDark)}`;
        }
    }

    function mouseDown(event:MouseEvent) {
        expand = true;
        oldPosOffset = event.offsetY;
    }

    function touchDown(event: TouchEvent) {
        expand = true;
        oldPosOffset = event.targetTouches[0].clientY - document.querySelector("#inner")!.getBoundingClientRect()!.top;
    }

    function moveMouse(event: MouseEvent) {
        if (!expand) return;
        top = Math.max(event.pageY-oldPosOffset, fromTop-userMinDragHeight);
    }

    function moveTouch(event: TouchEvent) {
        if (!expand) return;
        top = Math.max(event.targetTouches[0].clientY - oldPosOffset, fromTop-userMinDragHeight);
    }


    function mouseUp() {
        expand = false;

        if (top > (fromTop + closeHeight)) {
            close();
        } else {
            transitionTop = actuallAnimate;
            top = fromTop;
            setTimeout(() => transitionTop=false, actuallAnimate ? 150 : 0);
        }
    }

    onMount(() => {
        autoAnimate = !(window.matchMedia(`(prefers-reduced-motion: reduce)`) as any as boolean === true || window.matchMedia(`(prefers-reduced-motion: reduce)`).matches === true);

        setTimeout(() => {
            backgroundColor = "#00000081";
            translateY = "0%"
        }, actuallAnimate ? 1 : 0);
    });
</script>

<section
    class="outer {actuallAnimate ? "animate" : ""}"
    style="--backgroundColor:{backgroundColor};--translateY:{translateY};--topPos:{top}px;--topPosConst:{fromTop}px;{headerFont ? `--font:${headerFont}`: ""}"
>
    <button class="closeBackground" on:click={close} />
    <section class="inner {transitionTop && "transitionTop"}" id="inner">
        <div class="header">
            {#if headerTitle}
                <h2>{headerTitle}</h2>
            {/if}
            {#if showHeaderBar}
                <div class="bar"></div>
            {/if}
            {#if showCloseButton}
                <button class="closeButton" on:click={close} title="Close Drawer">
                    <svg xmlns="http://www.w3.org/2000/svg" width="30" height="30" fill="currentColor" class="bi bi-x" viewBox="0 0 16 16">
                        <path d="M4.646 4.646a.5.5 0 0 1 .708 0L8 7.293l2.646-2.647a.5.5 0 0 1 .708.708L8.707 8l2.647 2.646a.5.5 0 0 1-.708.708L8 8.707l-2.646 2.647a.5.5 0 0 1-.708-.708L7.293 8 4.646 5.354a.5.5 0 0 1 0-.708"/>
                    </svg>
                </button>
            {/if}
            {#if isDraggable}
                <!-- svelte-ignore a11y-no-static-element-interactions -->
                <div
                    class="slider {expand && "expand"} {showCloseButton && "closeButtonShowing"}"
                    role="none"
                    on:touchstart={touchDown}
                    on:touchmove={moveTouch}
                    on:touchend={mouseUp}
                    on:mousedown={mouseDown}
                    on:mousemove={moveMouse}
                    on:mouseleave={mouseUp}
                    on:mouseup={mouseUp}
                />
            {/if}
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
        height: calc(100% - var(--topPosConst));
        width: 100%;
        background-color: white;
        transform: translateY(var(--translateY));
        border-top-right-radius: 10px;
        border-top-left-radius: 10px;
        overflow: visible !important;
    }

    .rest {
        width: 100%;
        height: calc(100% - 40px);
        overflow-y: auto;
    }

    .outer.animate .inner.transitionTop {
        transition: top 0.15s linear;
    }

    .outer.animate .inner {
        transition: transform 300ms ease-out;
    }

    .closeBackground {
        width: 100%;
        height: var(--topPosConst);
        border: none;
        background: none;
        outline: none;
    }

    .header {
        width: 100%;
        height: 40px;
        position: relative;
        overflow: visible !important;
        display: flex;
        align-items: center;
        justify-content: flex-start;
        padding: 0 10px;
    }

    .header h2 {
        font-family: var(--font, sans-serif);
        font-size: 1.25rem;
        font-weight: bold;
    }

    .slider {
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 200;
    }

    .slider.closeButtonShowing {
        width: calc(100% - 40px);
    }

    .closeButton:hover svg {
        fill: #696969;
    }

    .closeButton {
        position: absolute;
        top: 0;
        right: 20px;
        width: 40px;
        height: 40px;
        border: none;
        background: none;
        cursor: pointer;
    }

    .slider.expand {
        transform: translateY(-600px);
        height: 3000%;
    }

    :global(*:has(.slider.slider.expand)) {
        overflow: hidden;
        -webkit-user-select: none;
        -khtml-user-select: none;
        -moz-user-select: none;
        -o-user-select: none;
        user-select: none;
        -webkit-user-drag: none;
        -khtml-user-drag: none;
        -moz-user-drag: none;
        -o-user-drag: none;
        user-drag: none;
        overscroll-behavior: none;
    }

    .bar {
        width: 50px;
        height: 6px;
        background-color: #cccccc;
        border-radius: 100vw;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }
</style>