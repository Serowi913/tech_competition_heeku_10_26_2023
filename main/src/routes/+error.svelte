<script>

    import { auth } from "$lib/firebase";
    import { onAuthStateChanged } from "firebase/auth";

    let userAuthed = false;

    onAuthStateChanged(auth, (user) => {
        if (user) {
            console.log("user is authenticated");
            userAuthed = true;
        } else {
            console.log("wer are very unauthenticated");
            userAuthed = false;
        }
        //userAuthed = user ? true : false; ---- Shorthand, but i'd rather have readability
    });

</script>

<h1>What even is that.</h1>

{#if userAuthed}
    <a href="/homepage/">Return to homepage.</a>
{:else}
    <a href="/">Return to login screen</a>
{/if}
