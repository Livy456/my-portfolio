<svelte:head>
    <title>Home</title>
</svelte:head>

<script>
    import projects from '$lib/projects.json';
    import Project from '$lib/Projects.svelte';
    
    // Description for 1st project- corporate investor data analysis
    // "I found that wealthy neighborhoods, such as Fenway, South Boston, and Waterfront had the 
    // highest corporate ownership rate and had a lot of college students residing in that neighborhood. 
    // Additionally, I found that there was a potential correlation between high corporate ownership rate 
    // and vacancy rate of housing properties in neighborhoods in the Boston area. I concluded that a 
    // significant portion of those neighborhoods were owned by college institutions and this caused an 
    // increase in the housing property for those neighborhood making the housing units less affordable. "

</script>

<style>
dl.home_page_stats {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 0.85rem 1rem;
    margin: 0.5rem 0 0;

    dt, dd {
        min-width: 0;
    }

    dt {
        font-size: 0.75rem;
        letter-spacing: 0.04em;
        text-transform: uppercase;
        color: var(--muted, gray);
    }

    dd {
        margin: 0 0 0.35rem;
        font-size: clamp(1.5rem, 4vw, 2.4rem);
        font-variant-numeric: tabular-nums;
    }
}

@media (min-width: 40rem) {
    dl.home_page_stats {
        grid-template-columns: repeat(4, minmax(0, 1fr));

        dt {
            grid-row: 1;
        }

        dd {
            grid-row: 2;
            margin: 0;
        }
    }
}
</style>

<h1> Olivia Dias</h1>
<div class="home-page-container">
    
    <img class="portfolio-img" src="images/undergrad_headshot.jpg" alt="This is me!" width="250px" height="350px"> 
    <p class="biography">I am a full stack software engineer at MIT Media Lab, building an <a href="http://autograder.media.mit.edu/" target="_blank" rel="noreferrer">agentic automatic grading platform</a>.</p>
</div>

<!-- Adds github information to home page -->
{#await fetch("https://api.github.com/users/Livy456") }
    <p>Loading...</p>
{:then response}
    {#await response.json()}
        <p>Decoding...</p>
        
    {:then data} 
        <section class="github_stats">
            <!-- <p>The data is { JSON.stringify(data) }</p> -->
            <h2>My GitHub Stats</h2>
            <dl class="home_page_stats">
                <dt>Followers:</dt>
                <dd>{data.followers}</dd>

                <dt>Following:</dt>
                <dd>{data.following}</dd>

                <dt>Public Repos:</dt>
                <dd>{data.public_repos}</dd>

                <dt>Public Gists:</dt>
                <dd>{data.public_gists}</dd>

            </dl>
            
        </section>
        
    {:catch error}
        <p class="error">
            Something went wrong: {error.message}
        </p>
        
    {/await}
{:catch error}
    <p class="error">
        Something went wrong: {error.message}
    </p>

{/await}

<h2>Latest Projects</h2>
<div class="projects">
    {#each projects.slice(0, 3) as p}
        <Project info={p} hLevel=3/>
    {/each}
</div>