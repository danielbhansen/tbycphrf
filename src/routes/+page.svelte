<script>
    import '../app.css';
    import Footer from './Footer.svelte';
</script>

<div class="section home-header">
    <h1>TBYC PHRF Calculator</h1>
</div>
<div class="section">
    <div class="selection">
        <a class="button" href="/tod">Time on Distance</a>
        <a class="button" href="/tot">Time on Time</a>
    </div>
</div>
<Footer />
<style>
    :global(body) {
        background-color: #EAEFEF;
    }
    .home-header.section {
        padding-top: 0;
    }
    h1 {
        margin: 0;
        color: #191923;
        border-top: 10px solid #333446;
        margin: 10px;
        padding: 10px 0;
    }
    .selection {
        display: flex;
        flex-direction: column;
        padding: 10vh 40px 20vh;
    }
    a {
        padding: 1.5em 2em;
        border-radius: 4px;
        margin-bottom: 2.5rem;
        background-color: #7F8CAA;
        color: white;
        display: inline-block;
        text-decoration: none;
        text-align: center;
    }

</style>