<script lang="ts">


    //TODO: Fix the win condition checker to only check with the tile that has just been added instead of with all tiles


    // Represents the current player (red or yellow)  ** set this to be randomized for fun **
    let turn = Math.random() < 0.5;
    // The slots that have been taken in the current game, initialized containing seven empty arrays
    let taken : string[][] = [[],[],[],[],[],[],[]];
    // Used to store the Game Board
    let board : HTMLDivElement;
    // Used to store the slots and which player has taken which slots
    let slots = genScriptSlots()
    // Whether or not a player has won the game yet
    let won = false;
    // The indicator for whos turn it currently is
    let turn_indicator : HTMLDivElement;

    // Creates a dictionary with values to represent every slot on the game board
    function genScriptSlots() {
        let ret = {}
        for (const column of [1, 2, 3, 4, 5, 6, 7]) {
            for (const row of [1, 2, 3, 4, 5, 6]) {
                ret[`${column}-${row}`] = "None"
            }
        }
        return ret
    }

    function clickSpace(event) {
        // Disallows players to click more spaces after someone has won the game
        if (won) return;

        // Event target (can either be the desired slot or the column)
        let target : HTMLDivElement = event.target;
        // The target type (either column or slot)
        let type = target.classList[0];

        // Parsing the target_column depending on what the type of target is
        let target_column : number;
        if (type == "column") target_column = parseInt(target.id)
        else target_column = parseInt(target.id.split("-")[0])

        // Returns if the current column is full (Stacked up to the top)
        if (taken[target_column - 1].length == 6) return;

        // Appends current item number to the column in array
        if (taken[target_column - 1].length == 0) taken[target_column - 1].push("1");
        else taken[target_column - 1].push((taken[target_column - 1].length + 1).toString());

        // Sets the bottom-most unselected slot to the color of the current player (red or yellow)
        board.children[target_column - 1].children[ 6 - taken[target_column - 1].length].style.backgroundColor = (turn) 
        ? "yellow" : "red";
        
        // Changes that slots stored player from none to whoever just clicked on it
        slots[`${target_column}-${taken[target_column -1 ].length}`] = (turn) ? "yellow" : "red"
        
        // Checks the board to see if anyone has won
        if (checkBoard()) return;
        let total_length = 0;
        for (const column of taken) total_length += column.length

        if (total_length == 7 * 6) {
            alert("there are no more playable spaces on the board, refresh to restart")
            won = true;
            return;
            
        }

        // Inverts the turn for the next player
        turn = !turn
        turn_indicator.style.backgroundColor = (turn) ? "yellow" : "red"
    }

    function checkBoard() {
        // Goes through every array inside of taken (Taken is Array<Array<*potentially empty* : string>)
        for (let i = 1; i <= taken.length; i++) {
            let column = taken[i - 1]
            
            // Goes through every item in that column (column may be empty)
            for (const slot of column) {
                let key = `${i}-${slot}`
                
                // If there is a win on any direction for this tile
                if (
                check_top(key) || 
                check_top_right(key) || 
                check_right(key) || 
                check_bottom_right(key) || 
                check_bottom(key) || 
                check_bottom_left(key) || 
                check_left(key) || 
                check_top_left(key)) {
                    if (turn) alert("The yellow player has won")
                    else alert("The red player has won")
                    won = true;
                    return true;
                }
            }
        }
        return false;
    }
    function check_top(key) : boolean {
        // If slot is along the top of the board
        if (key.split("-")[1] == "6") return false;

        // Setting variables for the current slot
        let column = parseInt(key.split("-")[0])
        let slot = parseInt(key.split("-")[1])
        let color = slots[key]

        // If there are not at least 3 slots above this one
        if (slot > 3) return false;

        // Returns false if all slots don't match in desired direction
        for (let i = 0; i < 4; i++) {
            if (!(slots[`${column}-${slot + i}`] == color)) return false;
        }
        
        // Returns true (meaning that the player has won)
        return true;
    }
    function check_top_right(key) : boolean {
        // If slot is along the top of the board
        if (key.split("-")[1] == "6") return false;
        // If slot is along the right of the board
        if (key.split("-")[0] == "7") return false;

        // Setting variables for the current slot
        let column = parseInt(key.split("-")[0])
        let slot = parseInt(key.split("-")[1])
        let color = slots[key]

        if (slot > 3) return false;
        if (column > 4) return false;

        for (let i = 0; i < 4; i++) {
            if (!(slots[`${column + i}-${slot + i}`] == color)) return false;
        }

        return true;
    }
    function check_right(key) : boolean {
        // If slot is along the right of the board
        if (key.split("-")[0] == "7") return false;

        // Setting variables for the current slot
        let column = parseInt(key.split("-")[0])
        let slot = parseInt(key.split("-")[1])
        let color = slots[key]

        if (column > 4) return false;

        for (let i = 0; i < 4; i++) {
            if (!(slots[`${column + i}-${slot}`] == color)) return false;
        }

        return true;
    }
    function check_bottom_right(key) {
        // If slot is along the bottom of the board
        if (key.split("-")[1] == "1") return false;
        // If slot is along the right of the board
        if (key.split("-")[0] == "7") return false;

        // Setting variables for the current slot
        let column = parseInt(key.split("-")[0])
        let slot = parseInt(key.split("-")[1])
        let color = slots[key]

        if (column > 4) return false;        
        if (slot < 4) return false;

        for (let i = 0; i < 4; i++) {
            if (!(slots[`${column + i}-${slot - i}`] == color)) return false;
        }

        return true;
    }
    function check_bottom(key)  : boolean {
        // If slot is along the bottom of the board
        if (key.split("-")[1] == "1") return false;

        // Setting variables for the current slot
        let column = parseInt(key.split("-")[0])
        let slot = parseInt(key.split("-")[1])
        let color = slots[key]

        // If there are not at least four slots below this one
        if (slot < 4) return false;

        // Returns false if all slots don't match in desired direction
        for (let i = 0; i < 4; i++) {
            if (!(slots[`${column}-${slot - i}`] == color)) return false;
        }

        // Returns true (meaning that the player has won)
        return true;
    }
    function check_bottom_left(key) : boolean {
        // If slot is along the bottom of the board
        if (key.split("-")[1] == "1") return false;
        // If slot is along the left of the board
        if (key.split("-")[0] == "1") return false;

        // Setting variables for the current slot
        let column = parseInt(key.split("-")[0])
        let slot = parseInt(key.split("-")[1])
        let color = slots[key]

        if (slot < 4) return false;        
        if (column < 4) return false;

        for (let i = 0; i < 4; i++) {
            if (!(slots[`${column - i}-${slot - i}`] == color)) return false;
        }

        return true;
    }
    function check_left(key) : boolean {
        // If slot is along the left of the board
        if (key.split("-")[0] == "1") return false;

        // Setting variables for the current slot
        let column = parseInt(key.split("-")[0])
        let slot = parseInt(key.split("-")[1])
        let color = slots[key]

        // If there are less then three slots to the left
        if (column < 4) return false;

        for (let i = 0; i < 4; i++) {
            if (!(slots[`${column - i}-${slot}`] == color)) return false;
        }
        
        return true;
    }
    function check_top_left(key) : boolean {
        // If slot is along the top of the board
        if (key.split("-")[1] == "6") return false;
        // If slot is along the left of the board
        if (key.split("-")[0] == "1") return false;

        // Setting variables for the current slot
        let column = parseInt(key.split("-")[0])
        let slot = parseInt(key.split("-")[1])
        let color = slots[key]

        if (column < 4) return false;
        if (slot > 3) return false;

        for (let i = 0; i < 4; i++) {
            if (!(slots[`${column + i}-${slot - i}`] == color)) return false;
        }

        return true;
    }
</script>

<!-- Screen Content Panel -->
<div class="h-screen">
    <div class="h-full w-full flex items-center justify-center">
    
        <div class="absolute w-[125px] h-[125px] border-black border-2 border-solid top-0 left-0 transition-colors duration-500 
        rounded-md text-center text-lg" 
        bind:this={turn_indicator} style="background-color: {(turn) ? "yellow" : "red"};">Turn</div>

        <!-- Game Board -->
        <div bind:this={board} class="w-[80%] h-[95%] bg-blue-600 flex float-left">
            
            <!-- Seven vertical div elements, with 1/7 width of parent (The game board) -->
            {#each [1, 2, 3, 4, 5, 6, 7] as column}
            <div id="{column.toString()}" class="column w-[14.28%] transition-colors duration-700 
                 hover:bg-blue-500 cursor-pointer" on:click={clickSpace}>

                <!-- Six Circles used to represent each slot for the board -->
                {#each [1, 2, 3, 4, 5, 6] as row}
                <div id="{column}-{row}" class="slot h-[100px] m-2 border-transparent rounded-[50%] bg-white
                     border-blue-900 border-4 border-solid">
                </div>
                {/each}
            </div>
            {/each}
        </div>
    </div>
</div>