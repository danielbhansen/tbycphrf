<script>
    import '../app.css';
    import Footer from './Footer.svelte';
</script>

<div class="home-header">
    <h1>TBYC PHRF Calculator</h1>
</div>
<div class="main section">
    <div class="selection">
        <a class="button" href="/tot">Time on Time</a>
        <a class="button" href="/tod">Time on Distance</a>
    </div>
</div>
<Footer />
<style>
    :global(body) {
        background-color: #EAEFEF;
    }
    .home-header {
        padding-top: 0;
    }
    .main.section {
        background: url('../lib/bg.jpg') no-repeat scroll 50% 50%;
        background-size: cover;
    }
    h1 {
        margin: 0;
        background: #333446;
        padding: 1em;
        font-size: 1rem;
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
        background-color: #EAEFEF;
        color: #333446;
        display: inline-block;
        text-decoration: none;
        text-align: center;

    }

</style>