<script>

    import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword } from "firebase/auth";
    import { auth, db } from "$lib/firebase";
    import { goto } from "$app/navigation";
    import { browser } from "$app/environment";
    import { setDoc, collection, doc, getDoc } from "firebase/firestore";


    let loggingIn = true;
    let logInError = "";
    let submittingLogIn = false;
    let submittingMessage = "";

    const validEmailEndings = [
        'gmail.com',
        'fcstu.org',
        'outlook.com',
        'yahoo.com',
        'hotmail.com',
        'icloud.com'
    ]

    // Signs out the user if they're logged in
    auth.signOut().then(() => {  });

    function logInSubmit (e) {
        let form = document.getElementById("log_in_form");
        let uemail = form.login_email_input.value;
        let upassword = form.login_password_input.value;

        // If the user_email and user_password inputs are not empty
        // (Gonna add the control for making sure that the text
        //  inputted is an email later)
        if (uemail && upassword) {
            submittingLogIn = true;
            submittingMessage = "Logging in...";

            signInWithEmailAndPassword(auth, uemail, upassword)
            .then((userCredential) => {
                const user = userCredential.user;
                localStorage.setItem("userId", user.uid);
                getDoc(doc(db, "users", user.uid)).then((fdoc) => {
                    localStorage.setItem("username", fdoc.data()['username']);
                    goto("/homepage", {replaceState: true});
                })
            }).catch((error) => {
                console.log(error.code);
                console.log(error.message);
                submittingLogIn = false;
                if (error.code == "auth/invalid-email") {
                    logInError = "Invalid Email.";
                } else if (error.code == "auth/invalid-login-credentials") {
                    logInError = "Invalid Login Credentials";
                }
            });
        } else {
            logInError = "Please fill out form."
        }
    }

    function signUpSubmit (e) {
        let form = document.getElementById("sign_up_form");
        let uusername = form.signup_username_input.value;
        let uemail = form.signup_email_input.value;
        let upassword = form.signup_password_input.value;
        if (!(uusername && uemail && upassword)) {
            logInError = "Please fill out form.";
        } else if (!uemail.includes('@')) {
            logInError = "Please enter a valid email address.";
        } else if (!(validEmailEndings.includes(uemail.split('@')[1]))) {
            logInError = "Email domain (name@domain.com) not accepted.";
        } else if (upassword.length < 8) {
            logInError = "Password too short (8 Characters minimum)"
        } else {

            submittingLogIn = true;
            submittingMessage = "Signing up...";

            createUserWithEmailAndPassword(auth, uemail, upassword)
            .then((userCredential) => {
                // Signed up 
                const user = userCredential.user;
                localStorage.setItem("userId", user.uid);
                localStorage.setItem("username", uusername)
                setDoc(doc(db, "users", user.uid), {
                    userId: user.uid,
                    username: uusername
                }).then(() => {
                    goto("/homepage");
                });
                // ...
            })
            .catch((error) => {
            });
        }
    }

    if (browser) {
        document.body.style.backgroundImage = "none"
        document.body.style.backgroundColor = "rgb(79, 79, 79)";
        
        //document.body.style.backgroundColor = "rgb(79, 79, 79)";
        //document.body.style.backgroundColor = 'rgb(241, 252, 255)';
    }

</script>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>「 Heeku 」</title>
</head>
<body>

    <div id="sign_in_div">
        {#if !submittingLogIn}
            {#if !loggingIn}

                <p style="font-size: 30px;">Sign Up</p>

                <form id="sign_up_form">

                    <div id="username">
                        <input class="form_inputs" type="text" name="signup_username_input" placeholder="Username" required>
                    </div>

                    <div id="email">
                        <input class="form_inputs" type="text" name="signup_email_input" placeholder="Email" required>
                    </div>

                    <div id="password">
                        <input class="form_inputs" type="password" name="signup_password_input" placeholder="Password" required>
                    </div>

                    <a href="/homepage" on:click|preventDefault={signUpSubmit}><button class="submit_button" id="sign_up_button">Sign Up</button></a>

                </form>

                <p class="error_message">{logInError}</p>

                <button on:click={() => loggingIn = true}>Already registered? Log in</button>

            {:else if loggingIn}

                <p style="font-size: 30px;">Log In</p>

                <form id="log_in_form">

                    <div id="email">
                        <input class="form_inputs" type="text" name="login_email_input" placeholder="Email" required>
                    </div>

                    <div id="password">
                        <input class="form_inputs" type="password" name="login_password_input" placeholder="Password" required>
                    </div>

                    <a href="/homepage" on:click|preventDefault={logInSubmit}><button class="submit_button" id="log_in_button">Log In</button></a>

                </form>

                <p class="error_message">{logInError}</p>

                <button on:click={() => loggingIn = false}>Need an account? Sign up</button>

            {/if}
        {:else}
            <p id="submitting_message">{submittingMessage}</p>
        {/if}

    </div>
    
</body>

<style>

    @import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@300&family=Varela+Round&display=swap');

    body {
        background-color: rgb(79, 79, 79);
        /*background-image: url('../img/blueprint.jpg');
        background-attachment: fixed;*/
        font-family: 'Courier New', Courier, monospace;
    }

    :global(body) {
        font-family: Fredoka;
    }
    
    #sign_in_div {
        margin-top: 200px;
        text-align: center;
    }

    .form_inputs {
        width: 300px;
        height: 30px;
    }

    .submit_button {
        margin-top: 20px;
        width: 300px;
        height: 50px;
        font-size: 15px;
    }

    .error_message {
        color: red;
    }

    #submitting_message {
        padding-top: 150px;
    }

</style>