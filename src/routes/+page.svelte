<svelte:head>
    <style>
        body {
            background: #121213;
            font-family: 'Roboto', sans-serif;
            overflow: hidden;
            margin: 0;
        }
    </style>
    <link href="https://fonts.googleapis.com/css?family=Roboto" rel="stylesheet">
</svelte:head>

<div class="center-container">
    <div class="login-container">
        <div class="login-inner-container">
            <div class="logo">
                <Logo />
            </div>
            <input 
                class={"username-field " + (emptyUsername ? "bad-username" : "")}
                placeholder="Username"
                bind:value={username}
                on:input={() => {emptyUsername = false;}}
            >
            <button class="join-button" on:click={signUp}>Join</button>
        </div>
    </div>
</div>

<script lang="ts">
    import Logo from "../components/logo.svelte";

    import { onMount } from "svelte";
    import { config } from "../config.js";

    let username = ""; // Binded
    let emptyUsername = false;

    async function signUp() {
        if (username === "") {
            emptyUsername = true;
            return;
        }
        console.log(`Signing up with username ${username}`);

        let url = new URL(`${config.API_BASE}/user/@me`);
        url.searchParams.append("username", username);
        
        let response = await fetch(url, {
            method: "POST"
        });

        let data = await response.json();

        let token = data.token;
        localStorage.setItem("login_token", token);
        localStorage.setItem("current_user_id", data.user.id);
        location.href = "/game";
    }

    onMount(() => {
        // Yeet already logged in users
        let token = localStorage.getItem("login_token");

        if (token !== null) {
            location.href = "/game";
        }

        console.log({token});
    });
</script>

<style>
    .center-container {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        width: 100vw;
    }
    .login-container {
        height: 50vh;
        width: 30vw;
        background: #181819;
    }
    .login-inner-container {
        margin: 20%;
    }
    .username-field {
        background: #121213;
        color: #fff;
        width: 100%;
        height: 3vh;
        border: 0;
        border-radius: 4px;
        text-align: center;
    }
    .bad-username {
        border: 2px solid #f00;
    }
    .join-button {
        background: #5865f2;
        border: 0;
        font-weight: bold;
        font-size: 1.5vh;
        width: 100%;
        margin-top: 2vh;
        border-radius: 4px;
        height: 5vh;
        color: #fff;
    }
    .join-button:hover {
        background: #525ee5;
    }
    .join-button:active {
        background: #4954cc;
    }
    .logo {
        height: 5vh;
        width: 5vw;
        margin: auto;
        margin-bottom: 8vh;
    }
</style>
