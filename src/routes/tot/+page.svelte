<script>
    import { onMount } from 'svelte';    
    import Header from '../Header.svelte';
    import Footer from '../Footer.svelte';
    import fleetData from '../inshore.json';
    import { preventDefault } from 'svelte/legacy';
    let windFactor = 550;
    let coefficient = 550;
    let fleetOptions = Object.keys(fleetData);
    let chosenFleet = $state('');
    let fleetArray = $state([]);
    let chosenBoat = $state('');
    let chosenBoatRating = $state('');
    let boatsArray = $state([]);
    let compArray = $state([]);
    let boatDeltasArray = $state([]);  
    let elapsedHours = $state('');
    let elapsedMinutes = $state('');
    let elapsedSeconds = $state('');
    let submittedSeconds = $state(0);
    let submittedTime = $state(0);
    let correctedBoatTOT = $state(0);
    let correctedBoatTime = $state(0);
    let correctedCompsTOT = $state([]);
    let compDiffs = $state([]);
    let correctedTOT, convertedDiff;

    onMount(() => {
        let sFleet = localStorage.getItem("totfleet");
        let sBoat = localStorage.getItem("totboat");
        let sHours = localStorage.getItem("tothours");
        let sMins = localStorage.getItem("totminutes");
        let sSecs = localStorage.getItem("totseconds");

        if (sFleet) {
            chosenFleet = sFleet;
            loadFleet(chosenFleet);
        }
        if (sBoat) {
            chosenBoat = sBoat;
            loadBoats();
        }
        if (sHours) {
            elapsedHours = sHours;
            elapsedMinutes = sMins;
            elapsedSeconds = sSecs;
            submitTime();
        }
    })

    function loadFleet(x) {
        localStorage.removeItem("tothours");
        localStorage.removeItem("totminutes");
        localStorage.removeItem("totseconds");
        localStorage.removeItem("totboat");
        chosenBoat = '';
        fleetArray = Object.entries(fleetData);
        fleetArray.forEach(fleet => {
            if (fleet[0] === x) {
                boatsArray = fleet[1];
            }
        })
        localStorage.setItem("totfleet", x);
        //console.log(boatsArray);
    }
    function loadBoats() {
        compArray = [];
        boatDeltasArray = [];
        elapsedHours = '';
        elapsedMinutes = ''; 
        elapsedSeconds = '';
        submittedTime = '';
        correctedBoatTime = '';
        //localStorage.removeItem("tothours");
        //localStorage.removeItem("totminutes");
        //localStorage.removeItem("totseconds");
        localStorage.setItem("totboat", chosenBoat);
        boatsArray.forEach(boat => {
            if (boat.name === chosenBoat) {
                chosenBoatRating = boat.rating;
            } else {
                compArray.push(boat);
            }
        })
        compArray.forEach(comp => {
            let boatDelta = calculateDeltas(comp.rating);
            boatDeltasArray.push(boatDelta);
        })
    }
    function calculateDeltas(rating) {
        let ratingDiff = rating - chosenBoatRating;
        //console.log(ratingDelta)
        let spmd = 60 * ratingDiff / (windFactor + rating)
        let spmdr = spmd.toFixed(2); 
        return spmdr;
    }
    function submitTime(e) {
        e.preventDefault();
        submittedTime = '';
        // push all the inputs into an array to better manage it
        let elapsedCombined = [];
        localStorage.setItem("tothours", elapsedHours);
        localStorage.setItem("totminutes", elapsedMinutes);
        localStorage.setItem("totseconds", elapsedSeconds);
        elapsedCombined.push(elapsedHours);
        elapsedCombined.push(elapsedMinutes);   
        elapsedCombined.push(elapsedSeconds);
        // prepend zeros to inputs with a single digit value
        elapsedCombined.forEach((num, index) => {
            if (num.length === 1) {
                elapsedCombined[index] = '0' + num;
            }
        })
        
        // convert to seconds
        submittedSeconds = (+elapsedCombined[0]) * 60 * 60 + (+elapsedCombined[1]) * 60 + (+elapsedCombined[2]);

        submittedTime = new Date(submittedSeconds * 1000).toISOString().substring(11, 19);
        
        calculateTOT();
    }
    function calculateTOT() {
        boatsArray.forEach(boat => {
            calculateCorrected(boat.rating);
            if (boat.name === chosenBoat) {
                correctedBoatTOT = correctedTOT;
                correctedBoatTime = new Date(correctedTOT * 1000).toISOString().substring(11, 19);
            } else {
                correctedCompsTOT.push(correctedTOT);
            }
        })
        correctedCompsTOT.forEach((t, index) => {
            const diff = correctedBoatTOT - t;
            unconvertDeltas(diff, compArray[index].rating);
            compDiffs.push(convertedDiff);
        })
    }
    function calculateCorrected(rating) {
        const tcf = coefficient / (windFactor + rating);
        return correctedTOT = submittedSeconds * tcf;
    }
    function unconvertDeltas(delta, rating) {
        // convert deltas back to 'chosen boat' time
        const tcf = coefficient / (windFactor + rating);
        return convertedDiff = delta / tcf;
    }
    $effect(() => {
        //$inspect("chosenBoat", chosenBoat);
        //$inspect("compArray", compArray);
        //$inspect("boatDeltasArray", boatDeltasArray);
    })

</script>

<Header />

<!-- Choose Fleet and Boat -->
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

<!-- Boat Details -->
{#if chosenBoat}
<div class="boatDetails section">
    <div class="chosenBoat">
        <h2>{chosenBoat}</h2>
        <p>Rating: {chosenBoatRating}</p>
    </div>

<!-- Competitors -->
    <div class="competitors">
        <h3>Seconds per minute deltas:</h3>
        <p><small>The number of seconds per minute of racing you need to be within of each competitor to maintain at least equal standing:</small></p>
        <table class="dpmtable"><thead><tr><th>Competitor</th><th>Rating</th><th>s/min delta</th></tr></thead><tbody>
        {#each compArray as comp, index}
            {#if (boatDeltasArray[index] < 0)}
                <tr><td> {comp.name}</td><td>{comp.rating}</td><td>stay within <strong>{Math.abs(boatDeltasArray[index])}</strong> s/min</td></tr>
            {:else if (boatDeltasArray[index] > 0)}
                <tr><td>{comp.name}</td><td>{comp.rating}</td><td>stay <strong>{boatDeltasArray[index]}</strong> s/min ahead</td></tr>
            {:else}
                <tr><td>{comp.name}</td><td>{comp.rating}</td><td>you have equal rating</td></tr>
            {/if}
        {/each}
        </tbody></table>
    </div>
</div>

    <div class="times section">
        <form onsubmit={submitTime}>
            <p>Enter Your Race Elapsed Time (hh:mm:ss): </p>
            <div class="digits">
                <input type="number" min="0" max="99" inputmode="numeric" pattern="[0-9]*" placeholder="00" name="elapsedHours" id="elapsedHours" required="" bind:value={elapsedHours}>
                <span class="colon">:</span>
                <input type="number" inputmode="numeric" pattern="[0-9]*" min="0" max="59" placeholder="00"
                name="elapsedMins" id="elapsedMins" required="" bind:value={elapsedMinutes}>
                <span class="colon">:</span>
                <input type="number" inputmode="numeric" pattern="[0-9]*"
                min="0" max="59" placeholder="00" name="elapsedSecs" id="elapsedSecs" required="" bind:value={elapsedSeconds}>
                <input type="submit" value="Calculate!">
            </div>
        </form>
      </div>

    {#if correctedBoatTime.length > 0}
        <div class="deltas section">
            <p>Submitted Time:</p><h2>{submittedTime}</h2><h3>Your corrected time is: <span class="corrected">{correctedBoatTime}</span></h3>
            {#each compArray as comp, index}
                {#if compDiffs[index] < 0} 
                    {@const diff = Math.abs(compDiffs[index])}
                    {@const diffhours = new Date(diff * 1000).toISOString().substring(11, 19)}
                    <p>You need to be within at least <strong>{diffhours}</strong> of <strong>{compArray[index].name}</strong>.</p>
                {:else if compDiffs[index] > 0}
                    {@const diffhours = new Date(compDiffs[index] * 1000).toISOString().substring(11, 19)}
                    <p>You need to beat <strong>{compArray[index].name}</strong> by <strong>{diffhours}</strong>.</p>
                {:else}
                    <p>You and <strong>{compArray[index].name}</strong> are even, you just need to cross the line ahead.</p>
                {/if}
            {/each}
        </div>
    {/if}
  
{/if}

<Footer />
