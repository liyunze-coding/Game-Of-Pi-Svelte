<script lang="ts">
    import { onMount } from "svelte";
    
    const PI = "3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367892590360011330530548820466521384146951941511609433057270365759591953092186117381932611793105118548074462379962749567351885752724891227938183011949129833673362440656643086021394946395224737190702179860943702770539217176293176752384674818467669405132000568127145263560827785771342757789609173637178721468440901224953430146549585371050792279689258923542019956112129021960864034418159813629774771309960518707211349999998372978049951059731732816096318595024459455346908302642522308253344685035261931188171010003137838752886587533208381420617177669147303598253490428755468731159562863882353787593751957781857780532171226806613001927876611195909216420198938095257201065485863278865936153381827968230301952035301852968995773622599413891249721775283479131515574857242454150695950829533116861727855889075098381754637464939319255060400927701671139009848824012858361603563707660104710181942955596198946767837449448255379774726847104047534646208046684259069491293313677028989";

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
    let showSequence = false;
    let gameOverState = false;
    let showStart = true;
    let digitsRemembered = 0;
    let highestScore = 0;
    const volume = 0.3;

    $: highestScore = Math.max(highestScore, digitsRemembered);

    const soundMap = {
        "1" : "/Game-Of-Pi/sound/vibraphone-key-0.mp3",
        "2" : "/Game-Of-Pi/sound/vibraphone-key-1.mp3",
        "3" : "/Game-Of-Pi/sound/vibraphone-key-2.mp3",
        "4" : "/Game-Of-Pi/sound/vibraphone-key-3.mp3",
        "5" : "/Game-Of-Pi/sound/vibraphone-key-4.mp3",
        "6" : "/Game-Of-Pi/sound/vibraphone-key-5.mp3",
        "7" : "/Game-Of-Pi/sound/vibraphone-key-6.mp3",
        "8" : "/Game-Of-Pi/sound/vibraphone-key-7.mp3",
        "9" : "/Game-Of-Pi/sound/vibraphone-key-8.mp3",
        "0" : "/Game-Of-Pi/sound/vibraphone-key-9.mp3",
        "." : "/Game-Of-Pi/sound/vibraphone-key-10.mp3",
    }

    const keys = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '', '0', '.'];

    let userSequence = ''; // The sequence of numbers the user has entered
    let n = 0; // The number of digits to show

    async function handleKeyClick(key: string) {
        if (showSequence || gameOverState) return;

        // @ts-ignore
        const audio = new Audio(soundMap[key]);
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
            const audio = new Audio(soundMap[sequence[i]]);
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
            <button on:click={() => startGameOverlay.style.visibility = "hidden"}>Start</button>
        </div>

        <div id="restart-game-div" class="overlay" bind:this={restartOverlay}>
            <button on:click={resetGameStats}>Restart</button>
        </div>
    </div>

    {#if showStart && !gameOverState}<button on:click={finishSequence}>That's all I remember for now</button>{/if}
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
            }
        }
    
        .keypad {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-gap: 1rem;
            // padding: 1rem;
            max-width: 400px;
            margin: 0 auto;

            font-size: 2.5rem;

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
                // border-radius: 0.75rem;
                cursor: pointer;
                transition: background-color 0.2s;

                width: 6rem;
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