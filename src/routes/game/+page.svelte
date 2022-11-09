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

<div class="content">
    <div class="game-container">
        <div class="game">
            {#if board !== null}
                {#each localBoard as row}
                    <div class="row">
                        {#each row as tile}
                            <div class="tile" on:click={() => {selectedTile = tile;}}>
                                <div class="players">
                                    {#if currentPlayer.position[0] === tile.position[0] && currentPlayer.position[1] === tile.position[1]}
                                        <div class="player current-player" on:click={() => {selectedPlayer = null}}>
                                            
                                        </div>
                                    {/if}
                                    <!-- Enemies -->
                                    {#each players as player}
                                        {#if player !== currentPlayer && player.position[0] == tile.position[0] && player.position[1] === tile.position[1]}
                                        <div class="player enemy-player" on:click={() => {selectedPlayer = player}}>
                                            
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
                        <div class="action-points">
                            <div class="action-point-container">
                                <ActionPointIcon />
                            </div>
                            <span class="action-points">{currentPlayer.action_points}</span>
                        </div>
                        <div class="health">
                            <div class="health-container">
                                <HealthIcon />
                            </div>
                            <span class="health">{currentPlayer.health}</span>
                        </div>
                    </div>
                {/if}
                <hr />
            </div>

            <!-- Upgrades section -->
            <div class="upgrades-section">
                <h1>Upgrades</h1>
                <div class="upgrades">
                    <div class="upgrade range-upgrade">
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
                        <div class="action-points">
                            <div class="action-point-container">
                                <ActionPointIcon />
                            </div>
                            <span class="action-points">{currentPlayer.action_points}</span>
                        </div>
                        <div class="health">
                            <div class="health-container">
                                <HealthIcon />
                            </div>
                            <span class="health">{currentPlayer.health}</span>
                        </div>
                    </div>

                    <h2>Actions</h2>
                    <div class="selected-player-actions">
                        <button class="gift-button">Gift</button>
                        <button class="attack-button">Attack!</button>
                    </div>
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
                            <button class="move-to-button" on:click={moveTo}>Move to</button>
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
                        <button class="debug-points-button" on:click={() => {
                            selectedTileDirection = getSelectedDirection();
                        }}>tile direction: {selectedTileDirection}</button>
                    {:else}
                        <p>Selected tile: null</p>
                    {/if}
                    <button class="debug-points-button" on:click={getDebugPoints}>Debug points</button>
                    <hr>
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

    // DEBUG
    let selectedTileDirection = null;

    async function updateGameState() {
        let token = localStorage.getItem("login_token");
        let currentUserId = localStorage.getItem("current_user_id");

        let response = await fetch(`${config.API_BASE}/user/all`);

        players = await response.json();

        currentPlayer = players.filter((player) => player.id == currentUserId)[0];

        response = await fetch(`${config.API_BASE}/board`);
        board = await response.json();

        let _localBoard = [];

        board.forEach((row, row_id) => {
            let tiles = row.filter((tile, tile_id) => {
                let distance = getDistance(currentPlayer.position, [row_id, tile_id]);

                // DEBUG
                if (distance < (currentPlayer.range + 2)) {
                    //console.log({row_id, tile_id, distance});
                }
                return distance < (currentPlayer.range + 2);
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
            return "UP";
        } else if (selectedTile.position[1] > currentPlayer.position[1]) {
            return "DOWN";
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
            console.error(`Could not move: ${details.error}`);
        }
        updateGameState()
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

    onMount(() => {
        // Sanity check
        let token = localStorage.getItem("login_token");
        
        if (token === null) {
            location.href = "/";
        }
        updateGameState();
        updateGameStateTaskId = setInterval(updateGameState, 10000);
    });

    onDestroy(() => {
        clearTimeout(updateGameStateTaskId);
    });

</script>

<style>
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
    .sidebar {
        height: 100vh;
        width: 20vw;
        background: #181819;
        display: flex;
        justify-content: center;
        align-items: center;
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
    .row {
        display: flex;
        gap: 5px;
    }
    .tile {
        background: #aaa;
        height: 5vh;
        width: 5vh;
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
    .attack-button {
        background: #d83c3e;
        color: #000;
    }
    .gift-button {
        background: #40D61A;
        color: #000;
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
    .move-to-button {
        background: #40D61A;
        color: #000;
    }
    .debug-points-button {
        background: #40D61A;
        width: 100%;
        height: 3vh;
        color: #000;
    }
</style>
