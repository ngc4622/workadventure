<script lang="ts">
    import { highlightedEmbedScreen } from "../../../Stores/EmbedScreensStore";
    import CamerasContainer from "../CamerasContainer.svelte";
    import CoWebsitesContainer from "../CoWebsitesContainer.svelte";
    import MediaBox from "../../Video/MediaBox.svelte";
    import { coWebsiteManager } from "../../../WebRtc/CoWebsiteManager";
    import { afterUpdate, onMount } from "svelte";
    import { mediaBreakpointDown } from "../../../Utils/BreakpointsUtils";

    function closeCoWebsite() {
        if ($highlightedEmbedScreen?.type === "cowebsite") {
            if ($highlightedEmbedScreen.embed.closable) {
                coWebsiteManager.closeCoWebsite($highlightedEmbedScreen.embed);
            } else {
                coWebsiteManager.unloadCoWebsite($highlightedEmbedScreen.embed);
            }
        }
    }

    function minimiseCoWebsite() {
        if ($highlightedEmbedScreen?.type === "cowebsite") {
            highlightedEmbedScreen.removeHighlight();
            coWebsiteManager.resizeAllIframes();
        }
    }

    function expandCoWebsite() {
        if ($highlightedEmbedScreen?.type === "cowebsite") {
            coWebsiteManager.goToMain($highlightedEmbedScreen.embed);
        }
    }

    afterUpdate(() => {
        if ($highlightedEmbedScreen) {
            coWebsiteManager.resizeAllIframes();
        }
    });

    let embedLeftBlock: HTMLDivElement;

    let displayCoWebsiteContainer = mediaBreakpointDown("lg");

    const resizeObserver = new ResizeObserver(() => {
        displayCoWebsiteContainer = mediaBreakpointDown("lg");

        if (!displayCoWebsiteContainer && $highlightedEmbedScreen && $highlightedEmbedScreen.type === "cowebsite") {
            highlightedEmbedScreen.removeHighlight();
        }
    });

    onMount(() => {
        resizeObserver.observe(embedLeftBlock);
    });
</script>

<div id="embed-left-block" bind:this={embedLeftBlock}>
    <div id="main-embed-screen">
        {#if $highlightedEmbedScreen}
            {#if $highlightedEmbedScreen.type === "streamable"}
                {#key $highlightedEmbedScreen.embed.uniqueId}
                    <MediaBox isHightlighted={true} streamable={$highlightedEmbedScreen.embed} />
                {/key}
            {:else if $highlightedEmbedScreen.type === "cowebsite"}
                {#key $highlightedEmbedScreen.embed.iframe.id}
                    <div
                        id={"cowebsite-slot-" + $highlightedEmbedScreen.embed.iframe.id}
                        class="highlighted-cowebsite nes-container is-rounded"
                    >
                        <div class="actions">
                            <button type="button" class="nes-btn is-primary expand" on:click={expandCoWebsite}>></button
                            >
                            <button type="button" class="nes-btn is-secondary minimise" on:click={minimiseCoWebsite}
                                >=</button
                            >
                            <button type="button" class="nes-btn is-error close" on:click={closeCoWebsite}
                                >&times;</button
                            >
                        </div>
                    </div>
                {/key}
            {/if}
        {/if}
    </div>

    {#if displayCoWebsiteContainer}
        <CoWebsitesContainer />
    {/if}
</div>

<CamerasContainer highlightedEmbedScreen={$highlightedEmbedScreen} />

<style lang="scss">
    #embed-left-block {
        display: flex;
        flex-direction: column;
        flex: 0 0 75%;
        height: 100%;
        width: 75%;
    }
    #main-embed-screen {
        height: 82%;
        margin-bottom: 3%;

        .highlighted-cowebsite {
            height: 100% !important;
            width: 96%;
            background-color: rgba(#000000, 0.6);
            margin: 0 !important;

            .actions {
                z-index: 200;
                position: relative;
                display: flex;
                flex-direction: row;
                justify-content: end;
                gap: 2%;

                button {
                    pointer-events: all;
                }
            }
        }
    }
</style>
