<script>
    import { page } from '$app/stores';
    
    let pages = [
        {url: "./", title:"Home"},
        {url: "./projects", title:"Projects"},
        {url: "./resume", title:"Resume"},
        {url: "https://github.com/Livy456", title:"Portfolio"},
        {url:"./meta", title:"Meta"},
        {url: "./contact", title:"Contact"}
        
    ]

    

</script>
<!-- { JSON.stringify($page) }       -->

<nav>

    {#each pages as p}
        <a class="navigation_bar" href={ p.url } class:current={ "." + $page.route.id === p.url } target= { p.url.startsWith("http") ? "._blank" : null }>{ p.title }</a>
    
    {/each}  
          
</nav>