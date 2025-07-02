<script>
    import { page } from '$app/state';
</script>

<header>
    <h1>
        {#if (page.url.pathname === '/tot')}
            Time on Time Calculator
        {:else if (page.url.pathname === '/tod')}
            Time on Distance Calculator
        {/if}
    </h1>
    <a href="/">Home</a>
</header>
