<script>
    import { onMount } from 'svelte';  
    import Header from '../Header.svelte';
    import Footer from '../Footer.svelte';
    import fleetData from '../inshore.json';

    let fleetOptions = Object.keys(fleetData);
    let chosenFleet = $state('');
    let fleetArray = $state([]);
    let chosenBoat = $state('');
    let chosenBoatRating = $state('');
    let boatsArray = $state([]);
    let compArray = $state([]);
    let raceDistance = $state(0);

    function loadFleet(x) {
        compArray = [];
        chosenBoat = '';
        fleetArray = Object.entries(fleetData);
        fleetArray.forEach(fleet => {
            if (fleet[0] === x) {
                boatsArray = fleet[1];
            }
        })
        localStorage.setItem("fleet", x);
        //console.log(boatsArray);
    }

    function loadBoats() {
        compArray = [];
        //boatDeltasArray = [];
        //correctedBoatTime = '';
        boatsArray.forEach(boat => {
            if (boat.name === chosenBoat) {
                chosenBoatRating = boat.rating;
            } else {
                compArray.push(boat);
            }
        })
        compArray.forEach(comp => {
            let boatDelta = calculateDeltas(comp.rating);
            //boatDeltasArray.push(boatDelta);
        })
    }
    function calculateDeltas(rating) {
       
    }    

</script>
<Header />
<div class="choices section">
    <h1>Time on Distance Calculator</h1>
    <div class="fleetOptions">
        <label for="fleetSelect">Choose Your Fleet: </label>
        <select if="fleetSelect" bind:value={chosenFleet} onchange={()=>loadFleet(chosenFleet)}>
            {#each fleetOptions as fleet}
                <option>{fleet}</option>
            {/each}
        </select>
    </div>

    {#if chosenFleet}
    <div class="boatOptions">
        <label for="boatSelect">Choose Your Boat: </label>
        <select id="boatSelect" bind:value={chosenBoat} onchange={loadBoats}>
            {#each boatsArray as boat} 
                <option>{boat.name}</option>
            {/each}
        </select>
    </div>
    {/if}
</div>

{#if chosenBoat}
    <div class="times section">
    <div class="chosenBoat">
        <h2>{chosenBoat}</h2>
    <p>Rating: {chosenBoatRating}</p>
    </div>
        <form>
            <label>Enter the Race Distance (nautical miles): </label><br>
            <input type="number" min="0" inputmode="numeric" pattern="[0-9]*" placeholder="00" required bind:value={raceDistance}>
        </form>
    </div>
{/if}

<Footer />

<style>
    form input[type="number"] {
        width: 3em;
    }
</style>