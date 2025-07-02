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
    let compTODArray = $state([]);
    let compDiffsArray = $state([]);
    let chosenBoatTOD = $state('');

    onMount(() => {
        let sFleet = localStorage.getItem("todfleet");
        let sBoat = localStorage.getItem("todboat");
        let sDistance = localStorage.getItem("toddistance");

        if (sFleet) {
            chosenFleet = sFleet;
            loadFleet(chosenFleet);
        }
        if (sBoat) {
            chosenBoat = sBoat;
            loadBoats();
        }
        if (sDistance) {
            raceDistance = sDistance;
            calculateTOD();
        }
    })


    function loadFleet(x) {
        compArray = [];
        chosenBoat = '';
        fleetArray = Object.entries(fleetData);
        fleetArray.forEach(fleet => {
            if (fleet[0] === x) {
                boatsArray = fleet[1];
            }
        })
       localStorage.setItem("todfleet", x);
    }

    function loadBoats() {
        compArray = [];
        compTODArray = [];
        compDiffsArray = [];
        raceDistance = '';
        //localStorage.removeItem("toddistance");
        localStorage.setItem("todboat", chosenBoat);

        boatsArray.forEach(boat => {
            if (boat.name === chosenBoat) {
                chosenBoatRating = boat.rating;
            } else {
                compArray.push(boat);
            }
        })
    }

    function calculateTOD(e) {
        e.preventDefault();
        compTODArray = [];
        compDiffsArray = [];
        localStorage.setItem("toddistance", raceDistance);
        chosenBoatTOD = chosenBoatRating * raceDistance;
        compArray.forEach(comp => {
            const compTOD = comp.rating * raceDistance;
            compTODArray.push(compTOD);
        })
        calculateDeltas();
    }  
    function calculateDeltas() {
        compTODArray.forEach(x => {
            compDiffsArray.push(x - chosenBoatTOD);
        })
    }

</script>
<Header />
<div class="choices section">
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
    <div class="boatDetails section">
        <div class="chosenBoat">
            <h2>{chosenBoat}</h2>
            <p>Rating: {chosenBoatRating}</p>
        </div>
            <div class="competitors">
        <h3>Fleet Competitors:</h3>
        <table class="dpmtable"><thead><tr><th>Competitor</th><th>Rating</th></tr></thead><tbody>
        {#each compArray as comp}  
            <tr><td>{comp.name}</td><td>{comp.rating}</td></tr>
        {/each}
        </tbody></table>
    </div>
    </div>
    <div class="times section">
        <form onsubmit="{calculateTOD}">
            <p>Enter the Race Distance (nautical miles):</p>
            <div class="digits">
                <input type="number" min="0" max="999" inputmode="numeric" pattern="[0-9]*" placeholder="00" bind:value={raceDistance} required="" step="0.1">
                <input type="submit" value="Calculate!">
            </div>
        </form>
    </div>
    {#if compTODArray.length > 0}
        <div class="deltas section">
            <p><small>Submitted Race Distance:</small></p><h2>{raceDistance}nm</h2>
            {#each compArray as comp, index}
                {#if compDiffsArray[index] < 0} 
                    {@const diff = Math.abs(compDiffsArray[index])}
                    {@const diffhours = new Date(diff * 1000).toISOString().substring(11, 19)}
                    <p>You need to be within at least <strong>{diffhours}</strong> of <strong>{compArray[index].name}</strong></p>
                {:else if compDiffsArray[index] > 0}
                    {@const diffhours = new Date(compDiffsArray[index] * 1000).toISOString().substring(11, 19)}
                    <p>You need to beat <strong>{compArray[index].name}</strong> by <strong>{diffhours}</strong></p>
                {:else}
                    <p>You and <strong>{compArray[index].name}</strong> are even, you just need to cross the line ahead.</p>
                {/if}
            {/each}
        </div>
    {/if}
{/if}




<Footer />

<style>
    form input[type="number"] {
        width: 3em;
    }
    .deltas.section h2 {
        margin-top: 0;
    }
</style>