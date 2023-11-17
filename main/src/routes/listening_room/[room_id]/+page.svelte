<script>
    import { db } from "$lib/firebase";
    import { collection, addDoc, onSnapshot, doc, deleteDoc, getDocs, updateDoc, arrayUnion, arrayRemove, getDoc } from "firebase/firestore";
    import { onMount } from "svelte";
    import pop_ids from "$lib/video_ids/pop_ids.json";

    import { page } from '$app/stores'
    const roomId = $page.params.room_id;

    updateDoc(doc(db, "room_" + roomId, "users_present"), {
        users: arrayUnion(localStorage.getItem("username"))
    });

    // getDocs(collection(db, "room_" + roomId)).then((snapshot) => {
    //     snapshot.docs.forEach((fdoc) => {
    //         if (fdoc.id != "users_present") {
    //             deleteDoc(doc(db, "room_" + roomId, fdoc.id));
    //         }
    //     });
    // });

    let videoWidth = 775;
    let username = localStorage.getItem("username");
    let allMessages = [];
    let usersPresent = [];
    let makeNewVideo = false;
    let disableSnapshot = false;

    let randomNum = Math.floor(Math.random() * pop_ids['ids'].length);

    let artist = pop_ids['ids'][randomNum]['artist'];
    let videoId = "MfqI-W_JRQQ"; //pop_ids['ids'][randomNum]['id'];

    function keyPressed(event) {
        if (event.key == "Enter") {
            messageSubmit();
        }
    }

    function messageSubmit() {
        let message = document.getElementById("chat_input").value;
        // If the message is empty, don't do anything
        if (message == "") {

        } else {
            // If there is a message (censoring would occur before this)
            addDoc(collection(db, "room_" + roomId), {
                username: username,
                message: document.getElementById("chat_input").value
            }).then(() => {
                // When the message is delivered, clear the input field
                document.getElementById("chat_input").value = "";
            });
        }
    }
    
    function onTabClose() {
        // Stop onSnapshot from updating
        disableSnapshot = true;

        // Remove current user from the users_present array
        updateDoc(doc(db, "room_" + roomId, "users_present"), {
            users: arrayRemove(localStorage.getItem("username"))
        });

        // If the room is now empty. . .
        if (usersPresent.length == 0) {
            // . . . clear the video_id field

            // ***Not sure if this works, but it's not really needed anyways***

            updateDoc(doc(db, "room_" + roomId, "video_id"), {
                id: ""
            });
        }

        // Get the users_present in the room
        getDoc(doc(db, "room_" + roomId, "users_present")).then((fdoc) => {
            // If no one is left in the room. . . 
            if (fdoc.data()['users'].length == 0) {
                // Get all the docs in the room collection
                getDocs(collection(db, "room_" + roomId)).then((snapshot) => {
                    // For each doc. . . 
                    snapshot.docs.forEach((fdoc) => {
                        // If the doc isn't the users_present or video_id doc
                        // *** If I end up making more essential docs like these,
                        //     I'll make a list instead of the names of docs to
                        //     exclude when deleting, but for now it makes more
                        //     sense to just check if the doc isn't these two ***
                        if (fdoc.id != "users_present" && fdoc.id != "video_id") {
                            deleteDoc(doc(db, "room_" + roomId, fdoc.id));
                        }
                    });
                }).then(() => {
                    return null;
                });
            }
        })


    };

    // Function for regenerating the iframe element when the video_id changes
    function createVideo() {
        let videoFrame = document.createElement("iframe");
        videoFrame.id = "video_frame";
        videoFrame.title = "YouTube: ";
        videoFrame.width = videoWidth;
        videoFrame.height = 445;
        videoFrame.src = "https://www.youtube.com/embed/" + videoId;
        document.getElementById("frame_span").appendChild(videoFrame);
        makeNewVideo = false;
    }

    // Creates the element(s) that displays the users in the room (a.k.a. "roommates")
    function createRoommate(name) {
        if (name != username) {
            let nr = document.createElement("span");
            nr.innerHTML = name;
            nr.className = "roommate";
            nr.id = name;
            document.getElementById("roommates").appendChild(nr);
        }
    }

    // When the page is loaded
    onMount(() => {
        videoWidth = window.screen.width / 2 - 20;
        let chatDiv = document.getElementById("chat");
        let rommates = document.getElementById("roommates");

        // Calls whenever data in the specified collection is modified
        onSnapshot(collection(db, 'room_' + roomId), (snapshot) => {
            // If snapshot isn't disabled
            if (!disableSnapshot) {
                snapshot.docs.forEach((fdoc) => {
                    if (fdoc.id == "users_present") {
                        // Nothing.
                    } else if (fdoc.id == "video_id") {
                        // If the video_id in the document doesn't match the one in this file,
                        // and it's not empty (if there's a new video id). . . 
                        if (fdoc.data()['id'] != videoId && fdoc.data()['id'] != "") {
                            // Set the videoId in this file to the new id
                            videoId = fdoc.data()['id'];
                            makeNewVideo = true;
                        } else if (fdoc.data()['id'] == "") {
                            // If the id in the document is empty, create a new id and set that as the id in the document
                            let randomNum = Math.floor(Math.random() * pop_ids['ids'].length);
                            let nvid = pop_ids['ids'][randomNum]['id'];
                            updateDoc(doc(db, "room_" + roomId, "video_id"), {
                                id: nvid
                            });
                        }

                        if (makeNewVideo) {
                            // If there's a video frame in the document already, delete it
                            if (document.getElementById("video_frame")) {
                                document.getElementById("video_frame").remove();
                            }
                            createVideo();
                        }

                    } else {
                        // Process the messages
                        if (!(allMessages.includes(fdoc.data()['message']))) {
                            // If the messages list doesn't include the new message,
                            // add the new message to the messages list and create a new
                            // message in the document
                            allMessages.push(fdoc.data()['message']);
                            let nm = document.createElement('p');
                            nm.innerHTML = "<strong>" + fdoc.data()['username'] + ": </strong>" + fdoc.data()['message'];
                            chatDiv.appendChild(nm);
                        }
                    }
                });
            }
        });

        // Calls whenever the users_present array changes
        onSnapshot(doc(db, "room_" + roomId, "users_present"), (snapshot) => {
            let snapUsers = snapshot.data()['users']; 

            // For each user_present in this file's array,
            // if it's not in the users_present in the doc
            // (snap users), that means that the user left
            // the room, so we delete them from the roommates
            // list (by getting the element by id and removing it)
            usersPresent.forEach((u) => {
                if (!snapUsers.includes(u)) {
                    document.getElementById(u).remove();
                }
            });
            
            // For each user_present in the doc's array,
            // if the users_present in this file does not
            // include a the user from the doc's array,
            // that means that someone joined, so we need
            // to add them to the roommates list
            snapUsers.forEach((u) => {
                if (!usersPresent.includes(u)) {
                    console.log("new user joined.");
                    createRoommate(u);
                    usersPresent.push(u);
                }
            });

        });

    });

</script>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>「 Apollo 」 | Room {roomId}</title>
</head>
<body>
    <div id="top_bar">
        <h1>Pollo</h1>
        <span id="return_wrapper"><a href="../homepage" on:click={onTabClose} data-sveltekit-replacestate><button id="return_button">Return</button></a></span>
    </div>

    <div id="content_wrapper">
        <div id="youtube_div">
            <span id="frame_span">
                <!-- <iframe title="YouTube:" id="video_frame" width="{videoWidth}" height="445" src="https://www.youtube.com/embed/{videoId}"></iframe> -->
                <!--                    https://www.w3schools.com/howto/howto_css_responsive_iframes.asp                     -->
            </span>

            <div id="song_data_div">
                <ul id="song_data_list">
                    <li>Artist: {artist}</li>
                    <!--
                    <li>Playing: "10 Minute Timer" by Random Person</li>
                    <li>Album: Greatest Hits</li>
                    <li>Released: 1989</li>
                    -->
                </ul>
            </div>

        </div>

        <div id="chat_div">
            <h1 id="chat_header">Chat:</h1>
            <div id="roommates_wrapper">
                <span>Roomate(s):</span>
                <div id="roommates">
                    <!-- <span class="roommate">Serowi913, </span>
                    <span class="roommate">Fiyafast</span> -->
                </div>
            </div>
            <div id="chat">
                <!-- <p><strong>Person1:</strong> This song is garbage</p> -->
            </div>
            <div id="chat_input_div">
                <button id="send_button" on:click={() => messageSubmit()}>Send:</button>
                <input id="chat_input" type="text" on:keydown={keyPressed}>
            </div>
            
        </div>
    </div>
</body>

<svelte:window on:beforeunload={onTabClose}/>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@300&family=Varela+Round&display=swap');

    body {
        font-family: Fredoka;
    }

    #top_bar {
        text-align: center;
    }

    #top_bar h1 {
        border-bottom: 2px solid rgb(230, 230, 230);
    }

    #return_wrapper {
        position: absolute;
        right: 0;
        top: 34px;
    }

    #return_button {
        border: none;
    }

    #content_wrapper {
        display: flex;
        flex-wrap: wrap;
    }

    #video_frame {
        border: 5px inset white;
    }

    #chat_div {
        margin-left: 10px;
        padding-left: 10px;
        flex-grow: 1;
        border: 2px solid whitesmoke;
    }

    #chat_header {
        border-bottom: 2px solid rgb(241, 241, 241);
        margin-top: 0px;
        margin-bottom: 0px;
        margin-right: 10px;
    }

    #roommates_wrapper {
        margin-top: 10px;
        margin-bottom: 10px;
    }

    #chat {
        position: absolute;
        max-height: 330px;
        overflow-y: auto;
        width: 45%;
        border: 1px solid rgb(220, 220, 220);
    }

    #chat_input_div {
        margin-top: 366px;
    }

    #chat_input {
        width: calc(100% - 70px);
    }

</style>

<!--
#youtube_div {
    height: 100%;
    width: 50%;
    float: left;
}

#chat_wrapper {
    position: absolute;
    left: 50%;
    float: left;
    width: 48%;
    margin-left: 8px;
    padding-left: 4px;
    padding-bottom: 320px;
    border-style: solid;
    border-width: 4px;
}

#chat_wrapper h1 {
    border-bottom-style: solid;
    border-bottom-width: 2px;
}

#chat {
    width: 200%;
    overflow-y: scroll;
    margin-left: 20px;
    max-width: 100%;
    max-height: 70%;
}

#chat_input {
    position: absolute;
    display: inline-block;
    bottom: 1%;
    left: 60px;
    width: 60%;
}

#send_button {
    position: absolute;
    display: inline-block;
    bottom: 1%;
}

#youtube_frame {
    width: 100%;
    border-style: none;
}

#song_data_div {
    background-color: lightpink;
}

#song_data_list {
    list-style-type: none;
    margin-left: -34px;
}

#song_data_list li {
    padding: 4px 0px 8px 0px;
}
-->