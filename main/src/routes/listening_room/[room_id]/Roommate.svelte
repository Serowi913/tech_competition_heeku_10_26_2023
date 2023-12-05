<script>

    export let username = "USERNAME";

</script>

<svelte:options accessors/>

<span class="roommate" id={username}>{username}</span>

<style>

    .roommate {
        padding: 2px 8px;
        border-radius: 10px;
        background-color: pink;
    }

</style>