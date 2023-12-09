<script>

    import { browser } from "$app/environment";
    import { goto } from "$app/navigation";
    import { auth, db } from "$lib/firebase";
    import { collection, doc, getDocs, setDoc } from "firebase/firestore";
    import { generate } from "$lib/globalFunctions";

    let creatingRoom = false;
    let joinCustomRoomError = "";

    function logOut() {
        auth.signOut().then(() => {
            goto("../");
        });
    }
    
    function joinCustomRoom() {
        let customRoomInput = document.getElementById("join_room_input");
        let customRoomId = customRoomInput.value;
        let errorMessage = document.getElementById("join_room_error_message");

        if (customRoomId.length < 8) {
            joinCustomRoomError = "Invalid room ID";
            errorMessage.hidden = false;
        } else {
            getDocs(collection(db, "room_" + customRoomId)).then((snapshot) => {
                if (snapshot.docs.length == 0) {
                    joinCustomRoomError = "Room does not exist";
                    errorMessage.hidden = false;
                } else {
                    goto("/listening_room/" + customRoomId)
                }
            });
        }
    }

    function createRoomSubmit() {
        let cusRoomId = 0;
        let createRoomForm = document.getElementById("create_room_form");
        if (createRoomForm.custom_id.value == "") {
            cusRoomId = generate(8);
        } else {
            cusRoomId = createRoomForm.custom_id.value;
        }

        setDoc(doc(db, "room_" + cusRoomId, "users_present"), {
            time: null,
            users: []
        }).then(() => {
            setDoc(doc(db, "room_" + cusRoomId, "video_id"), {
                time: null,
                id: ""
            }).then(() => {
                goto("/listening_room/" + cusRoomId)
            })
        });
    }

    if (browser) {
        document.body.style.backgroundImage = "none"
        document.body.style.backgroundColor = "rgb(79, 79, 79)";
    }

</script>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>「 Heeku 」 | Homepage</title>
</head>
<body>

    <header>
        <h1>Heeku</h1>
    </header>

    <div id="chatroom_links">
        <div class="room_link" id="room_1"><a href="/listening_room/1" data-sveltekit-replacestate><button class="room_button rb_a">Room I</button></a></div>
        <div class="room_link" id="room_2"><a href="/listening_room/2" data-sveltekit-replacestate><button class="room_button rb_b">Room II</button></a></div>
        <div class="room_link" id="room_3"><a href="/listening_room/3" data-sveltekit-replacestate><button class="room_button rb_a">Room III</button></a></div>
        <div class="room_link" id="room_4"><a href="/listening_room/4" data-sveltekit-replacestate><button class="room_button rb_b">Room IV</button></a></div>
        <div class="room_link" id="room_5"><a href="/listening_room/5" data-sveltekit-replacestate><button class="room_button rb_a">Room V</button></a></div>
        <div class="room_link" id="room_6"><a href="/listening_room/6" data-sveltekit-replacestate><button class="room_button rb_b">Room VI</button></a></div>
        <div class="room_link" id="room_7"><a href="/listening_room/7" data-sveltekit-replacestate><button class="room_button rb_a">Room VII</button></a></div>
        <div class="room_link" id="room_8"><a href="/listening_room/8" data-sveltekit-replacestate><button class="room_button rb_b">Room VIII</button></a></div>
        <div class="room_link" id="room_9"><a href="/listening_room/9" data-sveltekit-replacestate><button class="room_button rb_a">Room IX</button></a></div>
        <div class="room_link" id="room_10"><a href="/listening_room/10" data-sveltekit-replacestate><button class="room_button rb_b">Room X</button></a></div>
    </div>

    <div id="custom_room_options">
        <div id="join_room_wrapper">
            <label for="join_room_input">Join Room: </label>
            <input id="join_room_input" type="text"/>
            <button id="join_room_button" on:click={joinCustomRoom}>Join</button>
            <span id="join_room_error_message" hidden="true">{joinCustomRoomError}</span>
        </div>

        <div id="create_room_wrapper">
            <button on:click={() => {creatingRoom = true}}>Create room!</button>
        </div>

    </div>

    <div id="creating_room_wrapper" hidden={!creatingRoom}>

        <div id="creating_room">
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <svg on:click={() => {creatingRoom = false}} th="36px" height="48px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"><path fill-rule="evenodd" clip-rule="evenodd" d="M19.207 6.207a1 1 0 0 0-1.414-1.414L12 10.586 6.207 4.793a1 1 0 0 0-1.414 1.414L10.586 12l-5.793 5.793a1 1 0 1 0 1.414 1.414L12 13.414l5.793 5.793a1 1 0 0 0 1.414-1.414L13.414 12l5.793-5.793z" fill="#000000"></path></g></svg>
            <!--  https://www.svgrepo.com/svg/503004/close?edit=true  -->

            <h1>Create a Room!</h1>

            <form id="create_room_form" on:submit|preventDefault={createRoomSubmit}>

                <label for="custom_id">Custom Room ID (leave blank for random)</label>
                <input name="custom_id" type="number">

                <label for="genre_dropdown">Genre: </label>
                <select name="genre_select" id="genre_dropdown">
                    <option>Pop</option>
                    <option>Gospel Rock</option>
                </select>

                <button id="create_room_submit">Create room</button>

            </form>

        </div>

    </div>

    <div id="logout_wrapper"><button id="logout_button" on:click={logOut}>Log out</button></div>

</body>

<style>

    @import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@300&family=Varela+Round&display=swap');

    body {
        font-family: Fredoka;
    }

    button:hover {
        cursor: grabbing;
    }

    header {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        background-color: rgb(144, 0, 255);
        border-bottom: 7px solid rgb(120, 0, 200);
    }

    header h1 {
        text-align: center;
        color: white;
    }

    #chatroom_links {
        display: flex;
        /* width: 50%;
        margin-left: 25%; */
        justify-content: space-evenly;
        flex-wrap: wrap;
        align-content: flex-end;
        border-bottom: 2px solid black;
        padding-bottom: 20px;
        margin-top: 80px;
    }

    .room_link {
        flex: 1 1 0px;
    }

    .room_button {
        width: 100%;
        height: 50px;
        padding-top: 10px;
        padding-bottom: 10px;
        margin-top: 20px;
        border-radius: 5px;
    }

    .room_button.rb_a {
        border: 4px solid aqua;
    }

    .room_button.rb_b {
        border: 4px solid rgb(144, 0, 255);
    }

    #custom_room_options {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        margin-top: 50px;
    }

    #custom_room_options div {
        flex: 1 1 0px;
        max-width: 300px;
        height: 100px;
        text-align: center;
        border: 4px solid rgb(180, 180, 180);
        border-radius: 8px;
        padding: 10px;
        margin-left: 3px;
        margin-right: 3px;
        background-color: white;
    }

    #creating_room_wrapper {
        z-index: 10;
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.7);
    }

    #creating_room {
        margin: auto;
        width: auto;
        min-width: 50px;
        max-width: 600px;
        height: 400px;
        margin-top: 50px;
        background-color: white;
        border-radius: 10px;
        text-align: center;
    }

    #creating_room h1 {
        margin: auto 50px;
        border-bottom: 1px solid black;
    }

    #creating_room svg {
        position: relative;
    }

    #join_room_error_message {
        color: red;
        font-style: italic;
    }

    #logout_wrapper {
        margin-top: 10px;
        text-align: center;
    }

    #logout_button {
        width: 150px;
        height: 30px;
        border: 2px outset black;
    }

</style>