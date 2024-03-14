<script>
    // import '../../static/style.css';
    import Navbar from "../lib/Navbar.svelte";

    let colorScheme = "light dark";
    let root = globalThis?.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);

</script>
<style> 
label.color-scheme{
    position: absolute;
    top: 0.5em;
    right: 0.5rem;
    padding-left: 0.5rem;
    font-family: inherit;
}
</style>
<Navbar></Navbar>

<!-- <p>{colorScheme}</p> -->
<label class="color-scheme">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>

<slot />