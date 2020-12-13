<script>
    import { quintOut } from 'svelte/easing';
    import { crossfade } from 'svelte/transition';

    // prettier-ignore
    export let levels = [
        { countForMe: true,  drag: true,  balls: true,  singleDigitAnswer: true,  singleDigitInputs: true,  maxAnswer: 9 },
        { countForMe: false, drag: true,  balls: true,  singleDigitAnswer: true,  singleDigitInputs: true,  maxAnswer: 9 },
        { countForMe: false, drag: false, balls: true,  singleDigitAnswer: true,  singleDigitInputs: true,  maxAnswer: 9 },
        { countForMe: false, drag: false, balls: false, singleDigitAnswer: true,  singleDigitInputs: true,  maxAnswer: 9 },
        { countForMe: false, drag: false, balls: false, singleDigitAnswer: false, singleDigitInputs: true,  maxAnswer: 18 },
        { countForMe: false, drag: false, balls: false, singleDigitAnswer: false, singleDigitInputs: false, maxAnswer: 30 },
        { countForMe: false, drag: false, balls: false, singleDigitAnswer: false, singleDigitInputs: false, maxAnswer: 50 },
        { countForMe: false, drag: false, balls: false, singleDigitAnswer: false, singleDigitInputs: false, maxAnswer: 70 },
        { countForMe: false, drag: false, balls: false, singleDigitAnswer: false, singleDigitInputs: false, maxAnswer: 90 },
        { countForMe: false, drag: false, balls: false, singleDigitAnswer: false, singleDigitInputs: false, maxAnswer: 99 },
    ];
    export let questions = [];
    export let questionIndex = 0;
    export let levelIndex = 0;
    export let input = '';

    let level = levels[levelIndex];
    let question = questions[questionIndex];
    const balls = { lhs: [], rhs: [], ans: [] };
    $: onLevelChanged(levelIndex);
    $: onQuestionChanged(questionIndex);
    $: onInputChanged(input);

    const random = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;
    function onLevelChanged() {
        level = levels[levelIndex] || levels[levels.length - 1];
        questions = [];

        for (let i = 0; i < 10; i++) {
            let lhs, rhs, ans;

            if (level.singleDigitAnswer) {
                ans = random(2, 9);
                lhs = random(1, ans - 1);
                rhs = ans - lhs;
            } else if (level.singleDigitInputs) {
                lhs = random(1, 9);
                rhs = random(1, 9);
                ans = lhs + rhs;
            } else {
                ans = random(1, level.maxAnswer);
                lhs = random(1, ans - 1);
                rhs = ans - lhs;
            }

            if (questions.some((q) => q.lhs === lhs && q.rhs === rhs) === false) {
                questions.push({ lhs, rhs, ans });
            }
        }

        questionIndex = 0;
        question = questions[questionIndex];
    }

    function onQuestionChanged() {
        question = questions[questionIndex];
        balls.lhs = Array.from({ length: question?.lhs || 0 }, (_, i) => `lhs.${i}`);
        balls.rhs = Array.from({ length: question?.rhs || 0 }, (_, i) => `rhs.${i}`);
        balls.ans = [];
    }

    let timeout_inputChanged;
    function onInputChanged() {
        clearTimeout(timeout_inputChanged);
        timeout_inputChanged = setTimeout(() => {
            if (!input) return;
            if (parseInt(input) === question.ans) {
                input = '';
                questionIndex++;
                if (questionIndex === questions.length) {
                    levelIndex++;
                    speak(`Great job! Level ${levelIndex + 1}`);
                } else {
                    speak(`Correct!`, `That's right!`, `Good job!`);
                }
            } else {
                if (input.length === question.ans.toString().length) {
                    speak('Oops! Try again.');
                }
            }
        }, 250);
    }

    function onClickBalls() {
        if (!level.drag) return;

        const before = balls.ans.length;
        if (balls.lhs.length) balls.ans.push(balls.lhs.pop());
        else if (balls.rhs.length) balls.ans.push(balls.rhs.pop());

        if (balls.ans.length > before) {
            balls = balls;

            if (!level.countForMe) return;
            if (balls.ans.length === question.ans) speak(`${question.ans} balls`);
            else speak(balls.ans.length);
        }
    }

    function onClickKeypad({ target: el }) {
        if (el.matches('.keypad-btn-clear')) input = '';
        else if (el.matches('.keypad-btn')) input += el.innerText;
    }

    const [send, receive] = crossfade({
        duration: (d) => Math.sqrt(d * 200),

        fallback(node, params) {
            const style = getComputedStyle(node);
            const transform = style.transform === 'none' ? '' : style.transform;

            return {
                duration: 600,
                easing: quintOut,
                css: (t) => `transform: ${transform} scale(${t}); opacity: ${t}`,
            };
        },
    });

    function speak(...texts) {
        const text = texts[random(0, texts.length - 1)];
        const utterance = new SpeechSynthesisUtterance(text);
        speechSynthesis.speak(utterance);
    }
</script>

<style>
    :global(body) {
        display: flex;
        flex-direction: column;
        justify-content: space-around;

        background-color: #003687;
        color: rgb(110, 164, 245);
    }

    header,
    aside {
        font-family: 'Luckiest Guy';
        font-size: 10vh;
        text-align: center;
        padding: 0.5rem;
    }
    aside {
        font-size: 5vh;
        display: flex;
    }
    aside > div {
        flex: 1;
    }
    /* aside ~ aside {
        font-size: 3vh;
    } */
    main {
        display: grid;
        grid-template-columns: repeat(5, 1fr) 2fr;
        row-gap: 2vmin;
        padding: 5vmin 5vmin 2vmin;

        text-align: center;
    }

    .balls {
        grid-row: 1;
        height: 13vmin;
        gap: 1vmin;
        padding: 2vmin;
        box-shadow: inset 0 0 2px white;
        border-radius: 2vmin;

        display: flex;
        flex-flow: column wrap;
        justify-content: space-evenly;
        align-content: space-evenly;
        cursor: pointer;
    }
    .number,
    .operator {
        grid-row: 2;
        font-family: Pacifico;
        font-size: 20vmin;
    }
    .lhs {
        grid-column: 1;
    }
    .rhs {
        grid-column: 3;
    }
    .ans {
        grid-column: 5;
    }
    .keypad {
        grid-row: 2;
        grid-column: 6;
        align-self: center;
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        align-items: stretch;
        gap: 1vmin;
        padding: 2vmin;
    }
    .keypad-btn {
        all: unset;
        display: flex;
        align-items: center;
        justify-content: center;
        font-family: Pacifico;
        font-size: 4vmin;
        background-color: rgba(31, 46, 67, 0.5);
        cursor: pointer;
        border-radius: 0.5vmin;
        height: 10vmin;
    }

    .ball {
        border-radius: 50%;
        width: 3vmin;
        height: 3vmin;
        background-color: yellow;
        display: inline-block;
    }
</style>

<header>SUMS</header>

<aside>
    <div>Level {levelIndex + 1}</div>
    <div>Question {questionIndex + 1}</div>
</aside>

<!-- <aside>
    {#if level.countForMe}
        <div>Voice Counter</div>
    {:else if level.drag}
        <div>Draggable Counters</div>
    {:else if level.balls}
        <div>Counters</div>
    {/if}

    {#if level.singleDigitAnswer}
        <div>Single Digit Answers</div>
    {:else if level.singleDigitInputs}
        <div>Single Digit Inputs</div>
    {:else if level.maxAnswer}
        <div>Max Answer: {level.maxAnswer}</div>
    {/if}
</aside> -->

{#if question}
    <main>
        {#if level.balls}
            <div class="lhs balls" on:click={onClickBalls} role="button">
                {#each balls.lhs as key}
                    <div class="ball" in:receive={{ key }} out:send={{ key }} />
                {/each}
            </div>
            <div class="rhs balls" on:click={onClickBalls} role="button">
                {#each balls.rhs as key}
                    <div class="ball" in:receive={{ key }} out:send={{ key }} />
                {/each}
            </div>
            <div class="ans balls" on:click={onClickBalls} role="button">
                {#each balls.ans as key}
                    <div class="ball" in:receive={{ key }} out:send={{ key }} />
                {/each}
            </div>
        {/if}
        <div class="lhs number">{question.lhs}</div>
        <div class="plus operator">+</div>
        <div class="rhs number">{question.rhs}</div>
        <div class="equals operator">=</div>
        <div class="ans number">{input}</div>
        <div class="keypad" on:click={onClickKeypad}>
            <button class="keypad-btn">7</button>
            <button class="keypad-btn">8</button>
            <button class="keypad-btn">9</button>
            <button class="keypad-btn">4</button>
            <button class="keypad-btn">5</button>
            <button class="keypad-btn">6</button>
            <button class="keypad-btn">1</button>
            <button class="keypad-btn">2</button>
            <button class="keypad-btn">3</button>
            <button class="keypad-btn keypad-btn-clear" style="grid-column: 1 / 3">🙊</button>
            <button class="keypad-btn">0</button>
        </div>
    </main>
{/if}
