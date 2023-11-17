<script>

    import { browser } from "$app/environment";
    import { auth } from "$lib/firebase";

    if (browser) {
        document.body.style.backgroundColor = "white";
    }

</script>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>「 Apollo 」 | Homepage</title>
</head>
<body>

    <header>
        <h1>Apollo</h1>
    </header>

    <div id="chatroom_links">
        <div class="room_link" id="room_1"><a href="/listening_room/1" data-sveltekit-replacestate><button>Room I</button></a></div>
        <div class="room_link" id="room_2"><a href="/listening_room/2" data-sveltekit-replacestate><button>Room II</button></a></div>
        <div class="room_link" id="room_3"><a href="/listening_room/3" data-sveltekit-replacestate><button>Room III</button></a></div>
        <div class="room_link" id="room_4"><a href="/listening_room/4" data-sveltekit-replacestate><button>Room IV</button></a></div>
        <div class="room_link" id="room_5"><a href="/listening_room/5" data-sveltekit-replacestate><button>Room V</button></a></div>
        <div class="room_link" id="room_6"><a href="/listening_room/6" data-sveltekit-replacestate><button>Room VI</button></a></div>
        <div class="room_link" id="room_7"><a href="/listening_room/7" data-sveltekit-replacestate><button>Room VII</button></a></div>
        <div class="room_link" id="room_8"><a href="/listening_room/8" data-sveltekit-replacestate><button>Room VIII</button></a></div>
        <div class="room_link" id="room_9"><a href="/listening_room/9" data-sveltekit-replacestate><button>Room IX</button></a></div>
        <div class="room_link" id="room_10"><a href="/listening_room/10" data-sveltekit-replacestate><button>Room X</button></a></div>
    </div>

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
        border-bottom: 2px solid black;
    }

    header h1 {
        text-align: center;
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
    }

    .room_link {
        flex-grow: 1;
    }

    .room_link a button {
        width: 100%;
        height: 50px;
        padding-top: 10px;
        padding-bottom: 10px;
        margin-top: 20px;
    }


</style>