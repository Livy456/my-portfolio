<script>
    import Navbar from "../lib/Navbar.svelte";

    let localStorage = globalThis.localStorage ?? {};
    let colorScheme = "light dark";
    $: localStorage.colorScheme = colorScheme;
    // console.log??(localStorage.colorScheme);

    // prevents color scheme from going to 
    // default settings when page is reloaded
    if (localStorage.colorScheme) {
        colorScheme = localStorage.colorScheme;
    }

    let root = globalThis?.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);

    let profileDate = fetch("https://api.github.com/users/livy456");

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

<label class="color-scheme">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>

<slot />