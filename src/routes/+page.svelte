<script>
    import { onMount } from "svelte";
    import { fade } from "svelte/transition";
    import { cubicOut } from "svelte/easing";
    import { Howl, Howler } from "howler";

    let containerHeight;
    let panelHeight;
    let panels = [];
    let waterLevel;
    let start = false;
    let animationRate;
    let base;
    let incidentals;

    $: if (waterLevel) {
        panels = Array.from({ length: 144 }, () => ({ opacity: 0, color: randomGray() }));
    }

    $: if (start) {
        setTimeout(() => {
            base.play();
            base.fade(0, 1, 1000);
        });

        setTimeout(animate, 5000);
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

    function randomGray() {
        // random number between 3 and 7 inclusive
        const num = Math.floor(Math.random() * 5) + 3;
        return `#${num}${num}${num}${num}${num}${num}`;
    }

    function randomClick() {
        const chance = Math.random() * 50;

        if (chance < waterLevel) {
            const randomIndex = Math.floor(Math.random() * incidentals.length);
            const click = incidentals[randomIndex];

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

        //set animation rate to be between 900 and 1300 ms then multiply by the water level / 5
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

<div class="bg vh-100 overflow-hidden" bind:clientHeight={containerHeight}>
    {#if !waterLevel}
        <div class="w-100 position-absolute top-50 start-50 translate-middle d-flex flex-column align-items-center">
            <div class="spinner-border text-light" role="status">
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>
    {:else if !start}
        <div class="w-100 position-absolute top-50 start-50 translate-middle d-flex flex-column align-items-center" transition:fade>
            <h1 class="display-1 text-center mb-4">And Where Do I Seek The Dolphin Of Wisdom?</h1>
            <button class="btn btn-lg btn-outline-dark shadow-lg fs-2 fw-medium border border-3 border-dark px-5 py-3" transition:fade={{ easing: cubicOut }} on:click={() => (start = true)}>Start</button>
        </div>
    {:else}
        <div class="row" transition:fade={{ easing: cubicOut }}>
            {#each panels as panel}
                <div class="col-1 p-0" style="opacity: {panel.opacity}; transition: opacity {animationRate}ms cubic-bezier(0,.45,.9,1);">
                    <div class="panel w-100" style="height: {containerHeight / 12}px; background: {panel.color}; transition: background {animationRate}ms cubic-bezier(0,.45,.9,1);" />
                </div>
            {/each}
        </div>
    {/if}
</div>

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
