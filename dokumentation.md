# DOKUMENTATION 
this documentation is for educational purposes only and has therefore taken help of AI. 

___

## 1 HTML AND VIEW "RENDERING"
### 1.1 HTML CODE 
``` html 
    <main>
        <details class="addPlayer">
            <summary>Player details</summary>
            <form action="" method="get">
                <input type="text" name="name" placeholder="players name">
                <input type="submit" value="addPlayer">
            </form>
            <div class="players">

            </div>
        </details>
        <p class="whisperText">scroll horizontally to see all players</p>

        <div class="scoreTableDyn">

        </div>

        <div class="scoreTotal">

        </div>

        <button class="saveGameBtn">Save Game</button>
        <button class="startNewGameBtn">Start New Game</button>

        <details class="savedGamess">
            <summary>Saved Games</summary>
            <div class="savedGames">

            </div>
        </details>
    </main>
```
The HTML file relies on native *`details`* and *`summary`* tags to create collapsible 
sections which can be opened and closed on user command, this requires no javaScript 
to function. 
The file acts mainly as a skeleton, consisting of empty container elements. 
These act as placeholders where JavaScript will dynamically build and inject
future UI components. 
(*see 1.2 JAVASCRIPT HTML FUNCTIONS*).

___

### 1.2 JAVASCRIPT HTML FUNCTIONS
A lot of whats visible within the page is created with javascript (*see 3.2 CREATE ELEMENT* or *1.2a OUTPUTTING PLAYERS*).
#### 1.2a OUTPUTTING PLAYERS 
``` js
function playerInHtml(player) {
    const div = ce('div');
    div.classList.add('player');
    div.id = player.id;

    const title = ce('h3');
    title.innerText = player.name;

    const rmButton = ce('button');
    rmButton.innerText = "remove"
    rmButton.addEventListener('click', () => { removePlayer(player.id) })

    div.appendChild(title);
    div.appendChild(rmButton);
    return div;
}
``` 
This function accepts a single player object as an argument and builds a dedicated
DOM element (*`div`*) for each player. 

All nested elements are created via the function *`ce()`*, (*see 3.2 CREATE ELEMENT*). 
Elements are populated using the properties of the passed *player* object. 

For each player a remove button is created and attached to an event listener. 
It utilizes the specific player id, in order to know which player to delete upon 
clicks. 

The assembled player container is then returned so it can be added to the page. 

___

``` js
function printPlayers(players) {
    const playersBox = document.querySelector('.players');
    playersBox.replaceChildren();
    for (let player of players) {
        playersBox.appendChild(playerInHtml(player));
    }
}
```
The function *printPlayers* is responsible for rendering each player from the array 
of active players (*players*). <br>

First, it targets the empty *`.players`* container from the HTML. 
To ensure a reflection of current states without duplicating elements upon 
every update, *`replaceChildren()`* is called to clear the container. 

It then loops through the array of players, calling the *`playerInHtml()`* 
function for each one, and appends the returned elements to the page.  

___

#### 1.2b OUTPUTTING GAME INFO 
``` js 
function gameInfoHtml(court, players) {
    const infoDiv = ce('tr');
    infoDiv.classList.add('info');
    infoDiv.id = "courtId_" + court.id;

    const parNum = ce('td')
    parNum.innerText = court.par;

    const courtNum = ce('td')
    courtNum.innerText = court.id;

    infoDiv.appendChild(courtNum);
    infoDiv.appendChild(parNum);

    for (let player of players) {
        const playerScoreField = ce('td');
        const increasePlayerScore = ce('button');
        increasePlayerScore.innerText = '+';
        increasePlayerScore.addEventListener('click', () => { increaseScore(players, player.id, court.id) }) 
        const decreasePlayerScore = ce('button');
        decreasePlayerScore.innerText = '-';
        decreasePlayerScore.addEventListener('click', () => { decreaseScore(players, player.id, court.id, court.par) })

        const scoreSpan = ce('span');
        scoreSpan.innerText = player.scores[court.id] || 0;
        scoreSpan.style.margin = "0 6px";

        playerScoreField.appendChild(decreasePlayerScore);
        playerScoreField.appendChild(scoreSpan);
        playerScoreField.appendChild(increasePlayerScore);
        infoDiv.appendChild(playerScoreField);
    }
    return infoDiv;
}
``` 
This function generates a single table row (*`<tr>`*) representing one hole/court on 
the scorecard. 

It loops through the players array to generate a unique table cell for each 
participant. Inside this cell, "+" and "-" buttons are created and attached to 
listeners, which trigger functions of increase- and decrease score, passing along 
a player id and court id. 

The scoreSpan element: 
The *`<span>`* element is used as a lightweight, inline containers that
specifically holds the numeric score text. 
By applying a margin of '0 6px' to the span, it physically separates the 
"+" and "-" buttons. 
This prevents users from accidentally clicking the wrong button, especially on smaller
mobile screens where the table might require horizontal scrolling.  

___

``` js
async function printGameInfo(players) {
    let gameInfo = await getGameInfo();

    const infoBox = document.querySelector(".scoreTableDyn");
    infoBox.replaceChildren();

    const courtTitleRow = ce('tr');

    const parTitle = ce('th');
    parTitle.innerText = "par"

    const courtTitle = ce('th');
    courtTitle.innerText = "Hål";

    courtTitleRow.appendChild(courtTitle);
    courtTitleRow.appendChild(parTitle)

    const table = ce('table');
    table.appendChild(courtTitleRow);
    infoBox.appendChild(table);

    for (let player of players) {
        const playerTitle = ce('th')
        playerTitle.innerText = player.name;
        courtTitleRow.appendChild(playerTitle);
    }

    for (let court of gameInfo.court) {
        table.appendChild(gameInfoHtml(court, players));
    }
}
```
This function builds the main scorecard table. 

It first fetches the course data (*gameInfo*), selects the target HTML container and 
resets it. 

A header row is constructed for the output of columns for 'court', followed by a dynamic column for every player's name.
Finally, it loops through every court from the retrieved game data, calling
*`gameInfoInHtml()`* to generate and append scoring rows for each hole. 
___

#### 1.2c OUTPUTTING SCORES  

```` js 
function displaySavedGames() {
    const savedGamesDiv = document.querySelector(".savedGames");
    savedGamesDiv.replaceChildren();

    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i); 
     
        if (!key.startsWith("game_")) {
            continue;
        }

        const rawData = localStorage.getItem(key);  
        let gameData;
        try {
            gameData = JSON.parse(rawData); 
        } catch {
            gameData = rawData;
        }

        const singleGameDiv = ce("div");
        singleGameDiv.classList.add("savedGameItem");
        const savedGameTitle = ce("h4");
        savedGameTitle.innerText = key.replace("game_", "");

        const loadGameButton = ce("button");
        loadGameButton.innerText = "load Game";
        loadGameButton.addEventListener("click", () => {
            loadSavedGame(gameData);
        });
        const deleteSavedGameButton = ce("button");
        deleteSavedGameButton.innerText = "delete game";
        deleteSavedGameButton.addEventListener("click", () => {
            deleteSavedGame(gameData);
        });

        singleGameDiv.appendChild(savedGameTitle);
        singleGameDiv.appendChild(loadGameButton);
        singleGameDiv.appendChild(deleteSavedGameButton);
        savedGamesDiv.appendChild(singleGameDiv);
    }
    return savedGamesDiv;
}
```` 
The function first queries the container element for saved games from the DOM
and clears any existing children using *`replaceChildren()`* to prevent duplicates 
upon re-rendering. 

Because saved games persist in local Storage (*see 2.3a SAVE GAME FUNCTION*),
the function iterates through storage using a *for* loop. 
Although local Storage is a key-value store rather than an array, 
it still provides a *`.length`* property and a *`.key(index)`* method 
for inspection of entries by index. 

During each iteration, the key at the current index is checked; 
If it does not have the *`game_`* prefix, the iteration skips over it to ignore 
unrelated storage items (such as the currently active game). <br>
The stored string value for matchin keys is then retrieved via *`localStorage.getItem(key)`*
and assigned to the variable *`rawData`*. <br>
With the use of *`JSON.parse()`* an attempt of parsing *`rawData`* is done. 
If the parsing fails (i.e. the save is already stored as plain text), a *`try...catch`*
block catches the error and falls back to assigning the raw string to *`gameData`*.

___

## 2 JAVASCRIPT
### 2.1 PLAYERS 
#### ADD A PLAYER

To add a player, an event listener is attached to the html form *`.form`*.

``` js 
document.querySelector('.addPlayer form')
    .addEventListener('submit', e => {
        e.preventDefault();

        const name = e.target.name.value.trim().replaceAll(/\s+/g, "_");
        const scores = []; // default score to 0 if not provided 

        if (!name) return alert("even ghosts have names bro cmon");
        addPlayer(name, scores);
    })
```
*`e.preventDefault()`* stops the page from realoading upon submission. 
The input name is then retrieved, trimmed of whitespace, and has any spaces 
replaced with underscores. If left blank, the user is alerted. 
Otherwise, an empty scores array is initialized and passed to javascript. 
 

``` js
async function addPlayer(name, scores) {
    if (!name) return console.log("even ghosts have names cmon bro");

    const player = { name, scores, id: "id_" + Date.now() }

    const players = getPlayers() || [];

    players.push(player);
    saveToStorage(players);
    printPlayers(players);
    await updateGameViews(players); 
}
```
A new player object is constructed containing the obtained value of the name, an empty scores array, and a unique id generated with *`Date.now()`*. 

The current conten of players is retrieved via *`getPlayers()`*. 
The new player is pushed to this array, saved back to local storage and the DOM 
is told to re-render. 

The await keyword ensures that the program pauses execution here until 
*`updateGameViews()`* finished pulling external data and rendering 
the complex scorecard table. 

#### REMOVE PLAYER 
``` js
async function removePlayer(id) {
    const players = getPlayers();
    const newPlayerList = players.filter(p => p.id != id);
    if (players.length == newPlayerList.length) console.log("no player removed");
    saveToStorage(newPlayerList);
    printPlayers(newPlayerList);
    await updateGameViews(newPlayerList);
}
```


The function retrieves the specific id of the player targeted for removal. 
It uses the *`filter()`* method to create a *newPlayerList* containing only players
whose ID do not match the targeted one. 

It then compares the array lengths to verify that a deletion actually occurred, 
updates local storage, and triggers render functions to update the UI. 

___

### 2.2 PLAYER SCORE CONTROLS 
#### 2.2a INCREASE PLAYER SCORE 
``` js 
async function increaseScore(players, id, courtId) {
    const player = players.find(p => p.id == id);

    if (player.scores[courtId] === undefined || player.scores[courtId] === null) player.scores[courtId] = 0;
    player.scores[courtId] += 1;

    saveToStorage(players);
    await updateGameViews(players);
}
``` 
This function locates the correct player object by checking for an id match. 
It checks if a score for that specific court id exists yet; if it is undefined or null,
it initializes the score to 0. It then increments the score by 1, saves the mutated array to local storage and forces a UI update. 

#### 2.2b DECREASE PLAYER SCORE 
``` js 
async function decreaseScore(players, id, courtId) {
    const player = players.find(p => p.id == id);

    if (player.scores[courtId] === undefined || player.scores[courtId] === null) player.scores[courtId] = 0;
    if (!player.scores[courtId]) return;

    player.scores[courtId] -= 1;
    saveToStorage(players);
    await updateGameViews(players);
}
``` 
Similar to increasing the score, this finds the player and initializes empty
scores to 0. 

A guard clause (*`if(!player.scores[courtId]) return;`*) prevents the score from
dropping into negative numbers. 
If the score is above zero, it is decreased by 1 and the views are updated. 

#### 2.2c SHOW TOTALS ON TO THE PAGE 
``` js 
function showTotals(players, gameInfo) {
    const scoreTotal = document.querySelector(".scoreTotal");
    if (!scoreTotal) return null;
    const totalTitle = ce('h3');
    totalTitle.innerText = "Total Scores";

    scoreTotal.replaceChildren(totalTitle);
    const courtArray = gameInfo?.court || [];
    const playerStats = [];

    for (let player of players) { 
        const scores = player.scores || [];
        let playedScore = 0;
        let playedPar = 0;
        let playedHoles = 0;

        for (let hole of courtArray) {
            const score = scores[hole.id];
            if (score > 0) {
                playedScore += score;
                playedPar += hole.par;
                playedHoles++;
            }
        }
 
        let ratio = 1;
        if (playedPar > 0) {
            ratio = playedScore / playedPar;
        }
 
        let total = 0;

        for (let hole of courtArray) {
            const score = scores[hole.id];
            if (score > 0) {
                total += score;
            } else {
                total += Math.round(hole.par * ratio);
            }
        }

        playerStats.push({ name: player.name, total: total, playedHoles: playedHoles });
    }

    for (let player of playerStats) {
        const totalScoreForPlayerDiv = ce('div');
        const name = ce('h4');
        name.innerText = player.name;
        const totalScore = ce('p');
        totalScore.innerText = `Total score for ${player.name} : ${player.total}`;

        totalScoreForPlayerDiv.appendChild(name);
        totalScoreForPlayerDiv.appendChild(totalScore);
        scoreTotal.appendChild(totalScoreForPlayerDiv);
    }

    const validPlayers = playerStats.filter(p => p.playedHoles > 0);  
    const winnerTitle = ce('h3');

    if (validPlayers.length === 0) {
        winnerTitle.innerText = "No valid scores are recorded"
    } else {
        const lowestScore = Math.min(...validPlayers.map(p => p.total));
        
        const winners = validPlayers.filter(p => p.total === lowestScore); 
        if (winners.length > 1) {
            winnerTitle.innerText = `It's a tie between ${winners.map(p => p.name).join(", ")} with each their score being: ${lowestScore}`;
        } else {
            winnerTitle.innerText = `Winner: ${winners[0].name} with a total score of:  + ${winners[0].total}`;
        }
    }
    scoreTotal.appendChild(winnerTitle);
    return scoreTotal;
}
``` 
This function calculates the current standings and predicts final scores. 

For each player, it iterates through the holes they have actively played 
(*`score > 0`*). It compares their actual score against the total Par of those played
holes to generate a performance ratio (i.e. 12 strokes on holes where Par 
was 10, make a ratio of 1.2). 

It then loops through the entire course again. If a hole has a recorded score, its
added to the total. If a hole is unplayed, it predicts a score by multiplying the 
hole's Par by the player's performance ratio (*`Math.round(hole.par * ratio)`*). <br>
Finally, it filters out players who haven't played any holes, finds the lowest
total score using *`Math.min()`*, and outputs the winner (or a tie message) to the DOM. 

___

### 2.3 GAME CONTROLS 
#### 2.3a SAVE GAME FUNCTION
``` js 
function saveGame() {
    const currentGameInfo = localStorage.getItem('List');
    saveGameToLocalStorage(currentGameInfo);
}
``` 
The function extracts the active game state, which is temporarily stored under the
key 'List', and passes it to the permanent save helper. 

#### 2.3b LOAD THE SAVED GAME INFO 
``` js
async function loadSavedGame(gameData) {
    console.log(gameData);
    saveToStorage(gameData);
    printPlayers(gameData);
    await updateGameViews(gameData)
}
```
The function takes a retrieved game file, write it to the active 'List' storage key,
and re-renders the UI to display the loaded data. 

#### 2.3c START NEW GAME 

``` js
function startNewGame() {
    if (!confirm("Are you sure you want to start a new game?")) {
        return;
    }
    let players = getPlayers();
    for (let p of players) {
        p.scores = [];
    }
    loadSavedGame(players);
}
```
This function requires user confirmation to prevent accidental resets of a game. 

It loops through all current players, wiping their scores arrays clean and 
re-renders the blank slate. 

___

## 3 HELPER SECTION IN CLIENT.JS 
### 3.1 FETCHING & LOCALSTORAGE 
#### 3.1a LOCALSTORAGE SAVING 
``` js 
function saveToStorage(data) {
    const json = JSON.stringify(data); // gör om nå till en sträng
    localStorage.setItem('List', json);
}
``` 
This function updates the active game state. 

##### SAVE GAME TO LOCAL STORAGE 
``` js
function saveGameToLocalStorage(data) {
    let nameOfGame = null;
    while (true) {
        const input = prompt("Name your game: ")
        if (input === null) {
            return;
        }
        const trimName = input.trim();
        if (trimName.length > 0) {
            nameOfGame = trimName;
            break;
        }
        alert("even ghosts name their games bro cmon");
    }
    localStorage.setItem(`game_${nameOfGame}`, data);
}
```
This function creates a permanent save file using a user-propmpted name. 

#### 3.1b GAMEINFO 
``` js 
async function getGameInfo() {
    const jsonCard = await fetch("info.json");
    const card = await jsonCard.json();
    return card;
}
``` 
This function fetches the static course data (pars, hole, numbers) 
from a local JSON file. 

##### TOTAL SCORES HELPER
``` js
async function updateGameViews(players) {
    const gameInfo = await getGameInfo();
    printGameInfo(players);
    showTotals(players, gameInfo);
}
```
This is a bundle function that simultaneously refreshes both the main scorecard and 
the calculated totals section. 

#### 3.1c PLAYERS 
``` js 
function getPlayers() {
    let list = localStorage.getItem('List');
    if (!localStorage.length) return [];
    list = JSON.parse(list);
    return list
}
``` 
This retrieves the active game string from storage, 
parses it back into a javascript array, or returns an empty array 
if nothing exists. 

### 3.2 CREATE ELEMENT 
``` js 
function ce(elementType, className = null) {
    const el = document.createElement(elementType);
    if (className) el.classList.add(className);
    return el;
}
``` 
This function is a shorthand helper function that generates a DOM node 
and optionally assigns it a CSS class in one step, keeping the main code cleaner. 
