<!--
vim: ft=typescript
-->
<script lang="ts">
    import { gameManager } from "../../Phaser/Game/GameManager";
    import followImg from "../images/follow.svg";

    import { followStateStore, followRoleStore, followUsersStore } from "../../Stores/FollowStore";

    const gameScene = gameManager.getCurrentGameScene();

    function name(userId: number): string | undefined {
        return gameScene.MapPlayersByKey.get(userId)?.PlayerValue;
    }

    function sendFollowRequest() {
        gameScene.connection?.emitFollowRequest();
        followRoleStore.set("leader");
        followStateStore.set("active");
    }

    function acceptFollowRequest() {
        gameScene.CurrentPlayer.enableFollowing();
        gameScene.connection?.emitFollowConfirmation();
    }

    function abortEnding() {
        followStateStore.set("active");
    }

    function reset() {
        gameScene.connection?.emitFollowAbort();
        followUsersStore.stopFollowing();
    }

    function onKeyDown(e: KeyboardEvent) {
        if (e.key === "Escape") {
            reset();
        }
    }
</script>

<svelte:window on:keydown={onKeyDown} />

{#if $followStateStore === "requesting"}
    <div class="interact-menu nes-container is-rounded">
        {#if $followRoleStore === "follower"}
            <section class="interact-menu-title">
                <h2>Do you want to follow {name($followUsersStore[0])}?</h2>
            </section>
            <section class="interact-menu-action">
                <button type="button" class="nes-btn is-success" on:click|preventDefault={acceptFollowRequest}
                    >Yes</button
                >
                <button type="button" class="nes-btn is-error" on:click|preventDefault={reset}>No</button>
            </section>
        {:else if $followRoleStore === "leader"}
            <section class="interact-menu-question">
                <p>Should never be displayed</p>
            </section>
        {/if}
    </div>
{/if}

{#if $followStateStore === "ending"}
    <div class="interact-menu nes-container is-rounded">
        <section class="interact-menu-title">
            <h2>Interaction</h2>
        </section>
        {#if $followRoleStore === "follower"}
            <section class="interact-menu-question">
                <p>Do you want to stop following {name($followUsersStore[0])}?</p>
            </section>
        {:else if $followRoleStore === "leader"}
            <section class="interact-menu-question">
                <p>Do you want to stop leading the way?</p>
            </section>
        {/if}
        <section class="interact-menu-action">
            <button type="button" class="nes-btn is-success" on:click|preventDefault={reset}>Yes</button>
            <button type="button" class="nes-btn is-error" on:click|preventDefault={abortEnding}>No</button>
        </section>
    </div>
{/if}

{#if $followStateStore === "active" || $followStateStore === "ending"}
    <div class="interact-status nes-container is-rounded">
        <section class="interact-status">
            {#if $followRoleStore === "follower"}
                <p>Following {name($followUsersStore[0])}</p>
            {:else if $followUsersStore.length === 0}
                <p>Waiting for followers' confirmation</p>
            {:else if $followUsersStore.length === 1}
                <p>{name($followUsersStore[0])} is following you</p>
            {:else if $followUsersStore.length === 2}
                <p>{name($followUsersStore[0])} and {name($followUsersStore[1])} are following you</p>
            {:else}
                <p>
                    {$followUsersStore.slice(0, -1).map(name).join(", ")} and {name(
                        $followUsersStore[$followUsersStore.length - 1]
                    )} are following you
                </p>
            {/if}
        </section>
    </div>
{/if}

{#if $followStateStore === "off"}
    <button
        type="button"
        class="nes-btn is-primary follow-menu-button"
        on:click|preventDefault={sendFollowRequest}
        title="Ask others to follow"><img class="background-img" src={followImg} alt="" /></button
    >
{/if}

{#if $followStateStore === "active" || $followStateStore === "ending"}
    {#if $followRoleStore === "follower"}
        <button
            type="button"
            class="nes-btn is-error follow-menu-button"
            on:click|preventDefault={reset}
            title="Stop following"><img class="background-img" src={followImg} alt="" /></button
        >
    {:else}
        <button
            type="button"
            class="nes-btn is-error follow-menu-button"
            on:click|preventDefault={reset}
            title="Stop leading the way"><img class="background-img" src={followImg} alt="" /></button
        >
    {/if}
{/if}

<style lang="scss">
    .nes-container {
        padding: 5px;
    }

    div.interact-status {
        background-color: #333333;
        color: whitesmoke;

        position: relative;
        height: 2.7em;
        width: 40vw;
        top: 87vh;
        margin: auto;
        text-align: center;
    }

    div.interact-menu {
        pointer-events: auto;
        background-color: #333333;
        color: whitesmoke;

        position: relative;
        width: 60vw;
        top: 60vh;
        margin: auto;

        section.interact-menu-title {
            margin-bottom: 20px;
            display: flex;
            justify-content: center;
        }

        section.interact-menu-question {
            margin: 4px;
            margin-bottom: 20px;

            p {
                font-size: 1.05em;
                font-weight: bold;
            }
        }

        section.interact-menu-action {
            display: grid;
            grid-gap: 10%;
            grid-template-columns: 45% 45%;
            margin-bottom: 20px;
            margin-left: 5%;
            margin-right: 5%;
        }
    }

    .follow-menu-button {
        position: absolute;
        bottom: 10px;
        left: 10px;
        pointer-events: all;
    }

    @media only screen and (max-width: 800px) {
        div.interact-status {
            width: 100vw;
            top: 78vh;
            font-size: 0.75em;
        }

        div.interact-menu {
            height: 21vh;
            width: 100vw;
            font-size: 0.75em;
        }
    }
</style>
