<script>
    import { onMount } from "svelte";
    import { fade } from "svelte/transition";
    import { cubicOut } from "svelte/easing";
    import { Howl } from "howler";

    let containerHeight;
    let panelHeight;
    let panels = [];
    let waterLevel;
    let start = false;
    let animationRate;
    let base;
    let incidentals;
    let bgOpacity;
    let endPiece = false;

    $: if (waterLevel) {
        panels = Array.from({ length: 144 }, () => ({ opacity: 0, color: randomGray() }));
        bgOpacity = linearConversion(waterLevel, 2, 8, 0, 1);
    }

    $: if (start) {
        setTimeout(() => {
            base.play();
            base.fade(0, 1, 3000);
        });

        setTimeout(animate, 3000);
    }

    $: if (endPiece) {
        base.fade(1, 0, 10000);

        setTimeout(() => {
            location.reload();
        }, 10000);
    }

    async function fetchTides() {
        const res = await fetch("https://api.tidesandcurrents.noaa.gov/api/prod/datagetter?date=latest&station=9413450&product=water_level&datum=STND&time_zone=gmt&units=english&application=Forrest_Balman&format=json");
        const data = await res.json();

        if (res.ok) {
            waterLevel = data.data[0].v;
        } else {
            throw new Error(data);
        }
    }

    function linearConversion(oldValue, oldMin, oldMax, newMin, newMax) {
        const oldRange = oldMax - oldMin;
        const newRange = newMax - newMin;
        return ((oldValue - oldMin) * newRange) / oldRange + newMin;
    }

    function randomGray() {
        const num = Math.floor(Math.random() * 5) + 3;
        return `#${num}${num}${num}${num}${num}${num}`;
    }

    function randomClick() {
        const chance = Math.random() * (100 / waterLevel);

        if (chance < waterLevel && !endPiece) {
            const randomIndex = Math.floor(Math.random() * incidentals.length);
            const click = incidentals[randomIndex];

            click.volume(Math.random() * 0.5 + 0.5);
            click.seek(0);
            click.play();
        }
    }

    function animate() {
        for (let i = 0; i < panels.length; i++) {
            panels[i].opacity = Math.random() * 0.4;
            panels[i].color = randomGray();
        }

        panels = [...panels];

        animationRate = (Math.floor(Math.random() * 400) + 900) * (waterLevel / 5);
        randomClick();

        setTimeout(animate, animationRate);
    }

    onMount(() => {
        panelHeight = containerHeight / 12;

        fetchTides();

        base = new Howl({
            src: ["base.wav"],
            loop: true,
        });

        incidentals = [new Howl({ src: ["click-1.wav"] }), new Howl({ src: ["click-2.wav"] }), new Howl({ src: ["click-3.wav"] }), new Howl({ src: ["click-4.wav"] }), new Howl({ src: ["click-5.wav"] }), new Howl({ src: ["click-6.wav"] }), new Howl({ src: ["click-7.wav"] })];
    });
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
{#if !start}
    {#if !waterLevel}
        <div class="w-100 position-absolute top-50 start-50 translate-middle d-flex flex-column align-items-center">
            <div class="spinner-border text-light" role="status">
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>
    {:else}
        <div class="w-100 position-absolute top-50 start-50 translate-middle d-flex flex-column align-items-center" transition:fade>
            <h1 class="display-1 text-center text-light mb-4">And Where Do I Seek The Dolphin Of Wisdom?</h1>
            <button class="btn btn-lg btn-outline-light shadow-lg fs-2 fw-medium border border-3 border-light px-5 py-3" transition:fade={{ easing: cubicOut }} on:click={() => (start = true)}>Start</button>
        </div>
    {/if}
{:else}
    <div class="bg w-100 vh-100 position-absolute overflow-hidden" style="opacity: {bgOpacity};" bind:clientHeight={containerHeight} transition:fade={{ duration: 3000, easing: cubicOut }}>
        <div class="row">
            {#each panels as panel, i}
                <div class="col-1 p-0" style="opacity: {panel.opacity}; transition: opacity {animationRate}ms cubic-bezier(0,.45,.9,1);">
                    {#if i === 143}
                        <div class="panel w-100" style="height: {containerHeight / 12}px; background: {panel.color}; transition: background {animationRate}ms cubic-bezier(0,.45,.9,1); cursor: pointer;" on:click={() => (endPiece = true)} />
                    {:else}
                        <div class="panel w-100" style="height: {containerHeight / 12}px; background: {panel.color}; transition: background {animationRate}ms cubic-bezier(0,.45,.9,1);" />
                    {/if}
                </div>
            {/each}
        </div>
    </div>
{/if}

<style>
    .bg {
        background: linear-gradient(270deg, #8fffc7, #a985ff);
        background-size: 500% 500%;
        -webkit-animation: gradient 10s ease infinite;
        -moz-animation: gradient 10s ease infinite;
        animation: gradient 10s ease infinite;
    }

    @-webkit-keyframes gradient {
        0% {
            background-position: 0% 50%;
        }
        50% {
            background-position: 100% 50%;
        }
        100% {
            background-position: 0% 50%;
        }
    }

    @-moz-keyframes gradient {
        0% {
            background-position: 0% 50%;
        }
        50% {
            background-position: 100% 50%;
        }
        100% {
            background-position: 0% 50%;
        }
    }

    @keyframes gradient {
        0% {
            background-position: 0% 50%;
        }
        50% {
            background-position: 100% 50%;
        }
        100% {
            background-position: 0% 50%;
        }
    }
</style>
