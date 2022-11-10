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

<div class="logo-holder">
    <div class="logo-container">
        <Logo />
    </div>
</div>

<script>
    import Logo from "../../components/logo.svelte";
</script>

<style>
    .logo-holder {
        position: absolute;
        height: 100vh;
        width: 100vw;
        display: flex;
        justify-content: center;
        align-items: center;
    }

</style>
