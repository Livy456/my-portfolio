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

dl.home_page_stats{
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 1fr;

    dt{
        grid-row: 1;
        font-size: 20px;
    }

    dd{
        grid-row: 2;
        font-size: 50px;
    }
}
</style>

<h1> Olivia Dias</h1>
<div class="home-page-container">
    
    <img class="portfolio-img" src="images/SeniorPhoto.png" alt="This is me!" width="250px" height="350px"> 
    <p class="biography">I am a Junior at MIT, studying Artificial Intelligence and Decision Making.</p>
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
            <h2 style="font-family: garamond; font-size:30px;">My GitHub Stats</h2>
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