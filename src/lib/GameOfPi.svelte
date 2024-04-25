<script lang="ts">
    import { onMount } from "svelte";
    import Modal from "./Modal.svelte";
    
    const PI = "3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315";

    let showHowToPlay = false;
    let showCredits = false;

    // first digits until last 5 digits: fast
    // last 5 digits to last 3 digits: default
    // last 3 digits to last digit: slow
    const durations = {
        fast: 200,
        default: 400,
        slow: 600,
    }

    const sequence = PI.split('');
    let startGameOverlay: HTMLDivElement;
    let restartOverlay: HTMLDivElement;
    let showStartGameOverlay = true;
    let showSequence = false;
    let gameOverState = false;
    let showStart = true;
    let digitsRemembered = 0;
    let highestScore = 0;
    const volume = 0.3;

    $: highestScore = Math.max(highestScore, digitsRemembered);


    let soundMap = {
        "1" : new Audio("/game-of-pi/sound/vibraphone-key-0.mp3"),
        "2" : new Audio("/game-of-pi/sound/vibraphone-key-1.mp3"),
        "3" : new Audio("/game-of-pi/sound/vibraphone-key-2.mp3"),
        "4" : new Audio("/game-of-pi/sound/vibraphone-key-3.mp3"),
        "5" : new Audio("/game-of-pi/sound/vibraphone-key-4.mp3"),
        "6" : new Audio("/game-of-pi/sound/vibraphone-key-5.mp3"),
        "7" : new Audio("/game-of-pi/sound/vibraphone-key-6.mp3"),
        "8" : new Audio("/game-of-pi/sound/vibraphone-key-7.mp3"),
        "9" : new Audio("/game-of-pi/sound/vibraphone-key-8.mp3"),
        "0" : new Audio("/game-of-pi/sound/vibraphone-key-9.mp3"),
        "." : new Audio("/game-of-pi/sound/vibraphone-key-10.mp3"),
    }
    

    const keys = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '', '0', '.'];

    let userSequence = ''; // The sequence of numbers the user has entered
    let n = 0; // The number of digits to show

    function startModalClick() {
        startGameOverlay.style.visibility = 'hidden';
        showStartGameOverlay = false;
    }

    async function handleKeyClick(key: string) {
        if (showSequence || gameOverState) return;

        // @ts-ignore
        const audio = soundMap[key].cloneNode();
        audio.volume = volume;
        audio.play();
        
        userSequence += key;

        if (userSequence === sequence.join('').slice(0, userSequence.length)) {
            lightUpKey(key);
            digitsRemembered++;
        } else {
            lightUpKey(key, '#ff0000');
            gameOver();
        }

        if (userSequence.length === n) {
            if (userSequence === sequence.join('').slice(0, n)) {
                showSequence = true;
                lightUpAllKeys();
                await sleep(1000);
                nextRound();
            } else {
                console.log('Incorrect!');
            }

            userSequence = '';
        }
    }

    async function handleKeyDown(event: KeyboardEvent) {
        if (showSequence) return;
        const key = event.key;

        if (key === 'Enter') {
            if (gameOverState){
                resetGameStats();
            } else if (showStart) {
                finishSequence();
            }
        }

        if (gameOverState) return;

        if (keys.includes(key)) {
            handleKeyClick(key);
        }
    }

    function lightUpAllKeys() {
        document.querySelectorAll('.key').forEach((key: Element) => {
            let divKey = key as HTMLDivElement;
            divKey.style.backgroundColor = '#e0e0e0';
            divKey.style.color = '#e0e0e0';

            setTimeout(() => {
                divKey.style.backgroundColor = '#101010';
                divKey.style.color = '#e0e0e0';
            }, 200);
        });
    }

    function finishSequence() {
        n = userSequence.length + 1;
        userSequence = '';
        showStart = false;
        startGame();
    }

    function lightUpKey(keyChar:string, bgColor:string = '#e0e0e0', textColor:string = '#101010') {
        if (keyChar === '.') keyChar = 'dot';

        const key = document.getElementById(`key-${keyChar}`);
        if (key) {
            key.style.backgroundColor = bgColor;
            key.style.color = textColor;
            setTimeout(() => {
                key.style.backgroundColor = '#101010';
                key.style.color = '#e0e0e0';
            }, 200);
        }
    }

    async function sleep(ms: number) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    async function lightUpSequence(n: number, sequence: string[]){
        for (let i = 0; i < n; i++) {
            if (showSequence === false) return;

            // get duration
            let duration = durations.slow;

            let digitPos = n - i;

            if (digitPos >= 10) {
                duration = durations.fast;
            } else if (digitPos >= 5) {
                duration = durations.default;
            }
            
            // light up the PI digit
            lightUpKey(sequence[i]);

            // play the sound
            // @ts-ignore
            const audio = soundMap[sequence[i]].cloneNode();
            audio.volume = volume;
            audio.play();

            await sleep(duration);
        }
    }

    async function startGame() {
        showSequence = true;
        digitsRemembered = 0;
        await lightUpSequence(n, sequence);
        showSequence = false;
    }

    function nextRound() {
        n++;
        startGame();
    }

    function gameOver() {
        gameOverState = true;
        restartOverlay.style.visibility = 'visible';
    }

    function resetGameStats() {
        gameOverState = false;
        showSequence = false;
        userSequence = '';
        digitsRemembered = 0;
        n = 0;
        showStart = true;

        restartOverlay.style.visibility = 'hidden';
    }
    
    onMount(() => {
        window.addEventListener('keydown', handleKeyDown);

        return () => {
            window.removeEventListener('keydown', handleKeyDown);
        }
    });
</script>

<main>
    <Modal bind:showModal={showHowToPlay}>
        <h2>How to play</h2>
        <p>Remember the digits of Pi. Press the keys on your keyboard to enter the digits. Press Enter to start the game.</p>
    </Modal>

    <Modal bind:showModal={showCredits}>
        <h2>Credits</h2>
        <p>Created by <a href="https://twitter.com/raulcodes" target="_blank" rel="noopener noreferrer">@raulcodes</a></p>
    </Modal>

    <div id="heading">
        <div class="row-1">
            <h1>Game of Pi</h1>
            <div class="scores">
                <div class="score">
                    <span class="score-title">Current Digit</span>
                    <span class="score-value">{digitsRemembered}</span>
                </div>
                <div class="score">
                    <span class="score-title">Best</span>
                    <span class="score-value">{highestScore}</span>
                </div>
            </div>
        </div>
        <div class="row-2">
            <p class="tagline">Remember the digits of Pi</p>

            <button on:click={resetGameStats}>Restart</button>
        </div>
        <div class="row-3">
            <button on:click={() => showHowToPlay = true}>How to play</button>
            <button on:click={() => showCredits = true}>Credits</button>
        </div>
    </div>
    <div id="display-user-digits">
        {userSequence}
    </div>
    <div class="keypad">
        {#each keys as key}
            {#if key === "."}
                <button id={`key-dot`} 
                    class="key" 
                    on:click={() => handleKeyClick(key)} 
                >{key}</button>

            {:else if key !== ""}
                <button id={`key-${key}`} 
                    class="key" 
                    on:click={() => handleKeyClick(key)} 
                >{key}</button>

            {:else}
                <div class="empty"></div>
            {/if}
        {/each}

        <div id="start-game-div" class="overlay" bind:this={startGameOverlay}>
            <button on:click={startModalClick}>Start</button>
        </div>

        <div id="restart-game-div" class="overlay" bind:this={restartOverlay}>
            <button on:click={resetGameStats}>Restart</button>
        </div>
    </div>

    <div class="row-4">
        {#if showStart && !gameOverState && !showStartGameOverlay}<button on:click={finishSequence}>That's all I remember for now</button>{/if}
    </div>

    
</main>

<style lang="scss">
    $sm: 640px;
    $md: 768px;
    $lg: 1024px;
    $xl: 1280px;

    main {

        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;

        max-width: $xl;

        #heading {
            width: clamp(300px, 45%, 800px);
            .row-1 {
                display:flex;
                justify-content: space-between;
                align-items: start;
                height: fit-content;

                @media screen and (max-width: $lg) {
                    flex-direction:column;
                }

                h1 {
                    margin:0;
                }

                .scores {
                    display: flex;
                    flex-direction: row;

                    @media screen and (max-width: $lg) {
                        justify-content:center;
                        width: 100%;
                        margin-top: 1.25rem;
                    }

                    .score {
                        display: flex;
                        flex-direction: column;
                        align-items: center;
                        margin: 0 0.25rem;

                        border: solid 2px white;
                        padding: 0.25rem 1rem;
                        border-radius: 0.5rem;

                        .score-title {
                            font-size: 1rem;
                            font-weight: 500;
                        }

                        .score-value {
                            font-size: 1.5rem;
                            font-weight: 700;
                        }
                    }
                }
            }

            .row-2 {
                display: flex;
                flex-direction: row;
                justify-content: space-between; 
                align-items: center;

                @media screen and (max-width: $lg) {
                    flex-direction: column;
                    align-items: center;
                    margin-top: 0.5rem;
                    margin-bottom: 0.5rem;
                }

                .tagline {
                    font-size: 1.5rem;
                    font-weight: 500;
                    margin: 1rem 0;
                    text-align:left;
                }
                
                button {
                    border: solid 1px white;

                    &:hover {
                        background-color: white;
                        color: black;
                    }
                }
            }

            .row-3 {
                button {
                    border: solid 1px white;
                    font-size: 1.125rem;
                    line-height: 1.75rem;
                    padding-left: 1.25rem;
                    padding-right: 1.25rem;
                    padding-top: 0.5rem;
                    padding-bottom: 0.5rem;

                    transition-property: background-color, color;
                    transition-duration: 0.2s;
                    
                    &:hover {
                        background-color: white;
                        color: black;
                    }
                
                }
            }
        }

        .row-4 {
            position:relative;
            height: 3rem;
            margin-top: 1rem;
            margin-bottom: 1rem;

            button {
                border: #646cff solid 1px;
                
                &:hover {
                    background-color: #646cff;
                    color: white;
                }
            }
        }

        #display-user-digits {
            display: flex;
            justify-content: flex-end;
            font-size: 2rem;
            margin: 1rem 0;
            padding: 1rem 1rem;
            height: 3rem;
            border: solid 1px rgb(255 255 255);
            border-radius: 0.5rem;
            text-align: right;
            overflow: hidden;
            width: 50%;
        
            @media screen and (max-width: $md){
                width: 100%;
            }
        }
    
        .keypad {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 0.5rem;
            border-radius: 0.5rem;

            @media screen and (max-width: $md){
                gap: 0.5rem;
            }

            padding: 1rem 2rem;
            max-width: 400px;
            margin: 0 auto;

            font-size: 2.5rem;
            border-radius: 0.375rem;

            position: relative;

            border-radius: 0.375rem;

            .overlay#restart-game-div {
                visibility: hidden;
            }

            .overlay#start-game-div {
                visibility: visible;
            }

            .overlay {
                position: absolute;
                top: 0;
                left: 0;
                right: 0;
                bottom: 0;

                display: flex;
                justify-content: center;
                align-items: center;

                background: rgba(0, 0, 0, 0.5);

                button {
                    padding: 1rem 2rem;
                    font-size: 1.5rem;
                    font-weight: 700;
                    background-color: #101010;
                    color: white;
                    border: solid 2px white;
                    border-radius: 0.5rem;
                    cursor: pointer;
                    transition: background-color 0.2s;

                    z-index: 30;

                    &:hover {
                        background-color: #404040;
                    }
                }
            }

            .empty {
                visibility: hidden;
                height:0;
                width:0;
            }

            .key {
                display: flex;
                justify-content: center;
                align-items: center;
                background-color: #101010;
                border-radius: 0.75rem;
                cursor: pointer;
                transition: background-color 0.2s;

                width: 5.5rem;
                padding: 0;

                @media screen and (max-width: $md){
                    width: 5.5rem;
                    font-size: 2rem;
                }

                @media screen and (max-width: $sm){
                    width: 4rem;
                    font-size: 1.5rem;
                }

                aspect-ratio: 1/1;

                border: solid 2px white;

                user-select: none;

                &:hover {
                    background-color: #404040;

                }
            }
        }
    }
</style>