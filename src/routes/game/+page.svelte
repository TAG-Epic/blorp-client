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

<div class="notifications-container">
    {#each notifications as notification}
        <div class={"notification notification-type-" + notification.type} on:click={() => {removeNotification(notification);}}>
            <div class="notification-content-container">
                <h3 class="notification-title">{notification.title}</h3>
                <hr class="notification-devider" />
                <p class="notification-description">{notification.description}</p>
            </div>
        </div>
    {/each}
</div>

<div class="content">
    <div class="game-container">
        <div class="game">
            {#if board !== null}
                {#each localBoard as row}
                    <div class="row">
                        {#each row as tile}
                            {#if tile === row[0]}
                                <!-- Before first tile in row. Used for padding -->
                                {#if tile.position[0] - lowestX !== 0}
                                    {#each [...Array(tile.position[0] - lowestX)] as _}
                                        <div class="tile-padding"></div>
                                    {/each}
                                {/if}

                            {/if}
                            <div
                                class={getTileClasses(tile)}
                                on:click={() => {selectedTile = tile;}}
                                on:contextmenu|preventDefault={() => {
                                    if (getDistance(currentPlayer.position, tile.position) !== 1) {
                                        return;
                                    }
                                    selectedTile = tile;
                                    moveTo();
                                }}
                                title={tile.position.join(",")}
                            >
                                {#if getTileType(tile.tile_type) === "RESOURCEFUL"}
                                    <span class="tile-dig-resources">{tile.tile_type.RESOURCEFUL}</span>
                                {/if}
                                <div class="players">
                                    {#if currentPlayer.position[0] === tile.position[0] && currentPlayer.position[1] === tile.position[1]}
                                        <div class="player current-player" on:click={() => {selectedPlayer = null}}>
                                            
                                        </div>
                                    {/if}
                                    <!-- Enemies -->
                                    {#each players as player}
                                        {#if player !== currentPlayer && player.position[0] == tile.position[0] && player.position[1] === tile.position[1] && player.health !== 0}
                                        <div class="player enemy-player" on:click={() => {selectedPlayer = player}} title={player.username}>
                                            
                                        </div>
                                        {/if}
                                    {/each}
                                </div>
                            </div>
                        {/each}
                    </div>
                {/each}
            {/if}
        </div>
    </div>

    <div class="sidebar">
        <div class="inspector">
            <!-- Title section -->
            <div class="title-section">
                <h1>Inspector</h1>
                {#if currentPlayer !== null}
                    <div class="status-section">
                        <div class="action-points" title="Action points">
                            <div class="action-point-container">
                                <ActionPointIcon />
                            </div>
                            <span class="action-points">{currentPlayer.action_points}</span>
                        </div>
                        <div class="health" title="Health">
                            <div class="health-container">
                                <HealthIcon />
                            </div>
                            <span class="health">{currentPlayer.health}</span>
                        </div>
                        <div class="range" title="Range">
                            <div class="range-container">
                                <RangeIcon />
                            </div>
                            <span class="range">{currentPlayer.range}</span>
                        </div>
                    </div>
                {/if}
                <hr />
            </div>

            <!-- Upgrades section -->
            <div class="upgrades-section">
                <h1>Upgrades</h1>
                <div class="upgrades">
                    <div class="upgrade range-upgrade" on:click={() => {doUpgrade("range");}} title="Range upgrade">
                        <div class="icon-container">
                            <RangeIcon />
                        </div>
                    </div>
                </div>
                <hr>
            </div>

            <!-- Selected player section -->
            {#if selectedPlayer !== null}
                <div class="selected-player-section">
                    <h1>User: {selectedPlayer.username}</h1>
                    <div class="status-section">
                        <div class="action-points" title="Action Points">
                            <div class="action-point-container">
                                <ActionPointIcon />
                            </div>
                            <span class="action-points">{selectedPlayer.action_points}</span>
                        </div>
                        <div class="health" title="Health">
                            <div class="health-container">
                                <HealthIcon />
                            </div>
                            <span class="health">{selectedPlayer.health}</span>
                        </div>
                        <div class="range" title="Range">
                            <div class="range-container">
                                <RangeIcon />
                            </div>
                            <span class="range">{selectedPlayer.range}</span>
                        </div>
                    </div>

                    {#if getDistance(currentPlayer.position, selectedPlayer.position) <= currentPlayer.range}
                        <h2>Actions</h2>
                        <div class="selected-player-actions">
                            <button class="success-button" on:click={giftSelectedPlayer} title="Gift one action point">Gift</button>
                            <button class="danger-button" on:click={attackSelectedPlayer} title="Attack the player for 1 health. Costs 1 AP">Attack!</button>
                        </div>
                    {/if}
                    <hr />
                </div>
            {/if}

            <!-- Selected tile section -->
            {#if selectedTile !== null}
                <div class="selected-tile-section">
                    <h1>Selected tile</h1>
                    <h2>Actions</h2>
                    <div class="selected-tile-actions">
                        {#if getDistance(selectedTile.position, currentPlayer.position) === 1}
                            <button class="success-button" on:click={moveTo} title="Move to tile. Costs 1 AP">Move to</button>
                        {/if}
                        {#if getDistance(selectedTile.position, currentPlayer.position) === 0 && getTileType(selectedTile.tile_type) === "RESOURCEFUL"}
                            <button class="success-button" on:click={dig} title="Dig resource tile. Gives a random amount of AP">Dig</button>
                        {/if}
                    </div>
                </div>
                <hr />
            {/if}

            <!-- Debug section -->

            {#if config.DEBUG && currentPlayer !== null}
                <div class="debug-section">
                    <h1>Debug</h1>
                    <p>Username: {currentPlayer.username}</p>
                    <p>Range: {currentPlayer.range}</p>
                    <p>Position: {currentPlayer.position[0]}:{currentPlayer.position[1]}</p>
                    {#if selectedPlayer !== null}
                        <p>Selected player: {selectedPlayer.username}</p>
                    {:else}
                        <p>Selected player: null</p>
                    {/if}

                    {#if selectedTile !== null}
                        <p>Selected tile: {selectedTile.position[0]}:{selectedTile.position[1]}</p>
                        <button class="success-button" on:click={() => {
                            selectedTileDirection = getSelectedDirection();
                        }}>tile direction: {selectedTileDirection}</button>
                    {:else}
                        <p>Selected tile: null</p>
                    {/if}
                    <button class="success-button" on:click={getDebugPoints}>Debug points</button>
                    <button class="danger-button" on:click={deleteAccount}>Delete account</button>
                    <button class="success-button" on:click={() => {showMap = !showMap}}>Toggle map</button>
                    <button class="success-button" on:click={() => {
                        addNotification("info", "Test", "This is a test notification");
                    }}>Test notification</button>
                    <h3>Players</h3>
                    {#each players as player}
                        <p>{player.username} ({getDistance(player.position, currentPlayer.position)})</p>
                    {/each}
                </div>
            {/if}
        </div>
    </div>
</div>

<script>
    import ActionPointIcon from "../../components/action_point.svelte";
    import HealthIcon from "../../components/health.svelte";
    import RangeIcon from "../../components/range.svelte";

    import { onMount, onDestroy } from "svelte";
    import { config } from "../../config.js";

    let currentPlayer = null;
    let board = null;
    let localBoard = null;
    let players = null;
    let updateGameStateTaskId = null;
    let selectedPlayer = null;
    let selectedTile = null;
    let lowestX = 1000; // So high it will be changed later
    let notifications = [];
    let showMap = false;

    // DEBUG
    let selectedTileDirection = null;

    async function updateGameState() {
        let token = localStorage.getItem("login_token");
        let currentUserId = localStorage.getItem("current_user_id");


        let response = await fetch(`${config.API_BASE}/user/all`);

        if (!response.ok) {
            let details = await response.json();
            addNotification("error", "Failed to fetch players", details.error);
            return;
        }

        players = await response.json();
        
        /*players = players.map((player) => {
            player.action_points = calculateUpdatedPoints(player);
            return player;
        });
        */

        currentPlayer = players.filter((player) => player.id == currentUserId)[0];

        if (currentPlayer === undefined || currentPlayer === null) {
            console.log("Failed to find current player. Deleting account");
            deleteAccount();
            return;
        }

        // Death check
        if (currentPlayer.health === 0) {
            location.href = "/dead";
        }
        // Update selectedPlayer
        if (selectedPlayer !== null) {
            selectedPlayer = players.filter((player) => player.id == selectedPlayer.id)[0];
        }

        response = await fetch(`${config.API_BASE}/board`);

        if (!response.ok) {
            let details = await response.json();
            addNotification("error", "Failed to fetch board", details.error);
            return;
        }

        board = await response.json();
        board.reverse();

        lowestX = 1000; // Reset

        let _localBoard = [];

        board.forEach((row) => {
            let tiles = row.filter((tile) => {
                
                let distance = getDistance(currentPlayer.position, tile.position);

                if (distance > currentPlayer.range + 2 && !showMap) {
                    return false;
                }

                if (tile.position[0] < lowestX) {
                    lowestX = tile.position[0];
                }
                return true; 
            });
            if (tiles.length === 0) {
                return;
            }
            _localBoard.push(tiles);
        });

        localBoard = _localBoard; // State
    }

    function getDistance(pos1, pos2) {
        return Math.abs(pos1[0] - pos2[0]) + Math.abs(pos1[1] - pos2[1])
    }

    function getSelectedDirection() {
        console.log({currentPlayer: currentPlayer.position, selectedTile: selectedTile.position});
        if (selectedTile.position[0] > currentPlayer.position[0]) {
            return "RIGHT";
        } else if (selectedTile.position[0] < currentPlayer.position[0]) {
            return "LEFT";
        } else if (selectedTile.position[1] < currentPlayer.position[1]) {
            return "DOWN";
        } else if (selectedTile.position[1] > currentPlayer.position[1]) {
            return "UP";
        }

    }
    async function moveTo() {
        let direction = getSelectedDirection();
        console.log(`Moving ${direction}`);

        let token = localStorage.getItem("login_token");

        let response = await fetch(`${config.API_BASE}/board/move/${direction}`, {
            method: "POST",
            headers: {
                "authentication": token
            }
        });

        if (!response.ok) {
            let details = await response.json()
            addNotification("error", "Error while moving!", details.error);
            console.error(`Could not move: ${details.error}`);
            return;
        }
        updateGameState();
    }
    async function dig() {
        let token = localStorage.getItem("login_token");

        let response = await fetch(`${config.API_BASE}/board/dig`, {
            method: "POST",
            headers: {
                "authentication": token
            }
        });

        if (!response.ok) {
            let details = await response.json()
            addNotification("error", "Error while digging!", details.error);
            console.error(`Could not dig: ${details.error}`);
            return;
        }
        console.log(selectedTile);
        let points = selectedTile.tile_type.RESOURCEFUL;
        addNotification("info", "Dig", `Dug up ${points} action points`);
        updateGameState();
    }


    async function doUpgrade(upgradeType) {
        console.log(`Upgrading ${upgradeType}`);

        let token = localStorage.getItem("login_token");

        let response = await fetch(`${config.API_BASE}/user/@me/upgrade/${upgradeType}`, {
            method: "POST",
            headers: {
                "authentication": token
            }
        });

        if (!response.ok) {
            let details = await response.json()
            addNotification("error", "Error while upgrading!", details.error);
            console.error(`Could not upgrade: ${details.error}`);
            return;
        }
        addNotification("info", "Upgrade success", `Your ${upgradeType} upgrade has succeeded.`);
        updateGameState();
    }

    async function giftSelectedPlayer() {
        let token = localStorage.getItem("login_token");

        let response = await fetch(`${config.API_BASE}/user/${selectedPlayer.id}/gift`, {
            method: "POST",
            headers: {
                "authentication": token
            }
        });

        if (!response.ok) {
            let details = await response.json()
            addNotification("error", "Error while gifting!", details.error);
            console.error(`Could not gift: ${details.error}`);
            return;
        }
        addNotification("info", "Gift complete", `Gifted 1 action point to ${selectedPlayer.username}`);

        updateGameState();
    }
    async function attackSelectedPlayer() {
        let token = localStorage.getItem("login_token");

        let response = await fetch(`${config.API_BASE}/user/${selectedPlayer.id}/attack`, {
            method: "POST",
            headers: {
                "authentication": token
            }
        });

        if (!response.ok) {
            let details = await response.json()
            addNotification("error", "Error while attacking!", details.error);
            console.error(`Could not attack: ${details.error}`);
            return;
        }
        addNotification("info", "Attack complete", `Attacked ${selectedPlayer.username}`);
        updateGameState();
    }

    // Notifications
    function addNotification(type, title, description) {
        let notification = {
            type,
            title,
            description
        };
        notifications = notifications.concat([notification]);

        setTimeout(() => {
            removeNotification(notification);
        }, 5000);
    }
    function removeNotification(notification) {
        notifications = notifications.filter((checking_notification) => {
            return notification !== checking_notification;
        });
    }
    
    // Utils
    function calculateUpdatedPoints(player) {
        let creationDate = new Date(player.creation_date * 1000);
        let hours_since_creation_date = Math.abs(Math.floor((creationDate - new Date()) / 1000 / 60 / 60)); // Get hours
        hours_since_creation_date += 1; // Time zones suck. I do not want to deal with this.
        let todo_points = hours_since_creation_date - player.awarded_points;

        return player.action_points + todo_points;
    }

    function getTileType(tileType) {
        if (tileType === "EMPTY") {
            return tileType;
        }
        return Object.keys(tileType)[0];
    }

    // Debug

    async function getDebugPoints() {
        let token = localStorage.getItem("login_token");
        await fetch(`${config.API_BASE}/user/@me/points`, {
            method: "POST",
            headers: {
                "authentication": token
            }
        });
        updateGameState()
    }

    function deleteAccount() {
        localStorage.removeItem("login_token");
        location.href = "/";
    }

    function getTileClasses(tile) {
        let classes = ["tile"];
        
        let tileType = getTileType(tile.tile_type);
        classes.push("tile-type-" + tileType);

        if (getDistance(currentPlayer.position, tile.position) <= currentPlayer.range) {
            classes.push("tile-close-player");
        }
        players.forEach((player) => {
            if (player === currentPlayer) return;
            if (getDistance(player.position, tile.position) <= player.range && player.health !== 0) {
                classes.push("tile-close-enemy");
            }

        });

        return classes.join(" ");
    }

    onMount(() => {
        // Sanity check
        let token = localStorage.getItem("login_token");
        
        if (token === null) {
            location.href = "/";
        }
        updateGameState();
        updateGameStateTaskId = setInterval(updateGameState, 500);
    });

    onDestroy(() => {
        clearTimeout(updateGameStateTaskId);
    });

</script>

<style>
    /*
        Generic
    */
    span {
        color: #fff;
    }
    h1 {
        color: #aaa;
        text-align: center;
    }
    h2 {
        color: #aaa;
        text-align: center;
    }
    h3 {
        color: #aaa;
        text-align: center;
    }
    p {
        color: #fff;
    }
    button {
        border: 0;
        color: #ccc;
    }
    .content {
        display: flex;
    }
    .danger-button {
        background: #d83c3e;
        color: #000;
        width: 100%;
        height: 3vh;
    }
    .success-button {
        background: #40D61A;
        width: 100%;
        height: 3vh;
        color: #000;
    }


    /*Game*/
    .game-container {
        height: 100vh;
        width: 80vw;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .game {
        display: flex;
        gap: 5px;
        flex-direction: column;
    }
    .row {
        display: flex;
        gap: 5px;
    }
    .players {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100%;
        width: 100%;
    }
    .player {
        border-radius: 50%;
        height: 1.5vh;
        width: 1.5vh;
    }
    .current-player {
        background: #40D61A;
    }
    .enemy-player {
        background: #f00;
    }

    /* Tiles */
    .tile {
        background: #aaa;
        height: 2vh;
        width: 2vh;
    }
    .tile-type-RESOURCEFUL {
        background: #0FF;
    }
    .tile-padding {
        height: 2vh;
        width: 2vh;
    }
    .tile-close-player {
        background: #888;
    }
    .tile-close-enemy {
        background: #F88;
    }
    .tile-dig-resources {
        color: #000;
    }
    
    /*Inspector*/
    .sidebar {
        height: 100vh;
        width: 20vw;
        background: #181819;
        display: flex;
        justify-content: center;
        align-items: center;
        overflow: scroll;
    }
    .inspector {
        height: 98vh;
        width: 18vw;
    }
    .status-section {
        display: flex;
        justify-content: center;
    }
    .status-section > div {
        margin: 5px;
    }
    .action-point-container {
        float: left;
        height: 20px;
        width: 20px;
        margin-right: 5px;
    }
    .health-container {
        float: left;
        height: 20px;
        width: 20px;
        margin-right: 5px;
    }
    .range-container {
        float: left;
        height: 20px;
        width: 20px;
        margin-right: 5px;
    }
    .upgrades {
        display: flex;
        justify-content: center;
    }
    .range-upgrade > .icon-container {
        width: 20vw;
        padding: 15px;
    }
    .upgrade {
        height: 100px;
        width: 100px;
        display: flex;
    }
    .selected-player-actions > button {
        width: 100%;
        height: 3vh;
    }
    .selected-player-actions {
        display: flex;
        gap: .1vh;
    }
    .selected-tile-actions {
        display: flex;
        gap: .1vh;
    }
    .selected-tile-actions > button {
        width: 100%;
        height: 3vh;
    }

    /*Notifications*/
    .notifications-container {
        position: absolute;
        top: 1vh;
        left: 1vw;
        height: 100vh;
        width: 17vw;
        display: flex;
        flex-direction: column;
        gap: 5px;
        pointer-events: none;
    }
    .notification {
        width: 17vw;
        height: 20vh;
        display: flex;
        justify-content: center;
        align-items: center;
        pointer-events: initial;
    }
    .notification-type-info {
        background: #3c3c3f;
        border-style: rounded;
        border-radius: 20px;
    }
    .notification-type-error {
        background: #3c3c3f;
        border-radius: 5px;
        border-style: solid;
        border-color: transparent;
        border-left-color: #f00;
    }
    .notification-content-container {
        width: 80%;
        height: 90%;
    }
    .notification-title {
        color: #fff;
    }
</style>
