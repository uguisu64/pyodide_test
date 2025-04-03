<script>
    import { loadPyodideInstance } from "$lib/loadPyodide";
    import { onMount } from "svelte";

    let pyodide

    let in1 = $state(0)
    let in2 = $state(0)
    let res = $derived.by(async() => {
        return calcAdd(in1, in2)
    })

    onMount(async() => {
        pyodide = await loadPyodideInstance()
    })

    async function calcAdd(in1, in2) {
        pyodide = await loadPyodideInstance()
        let context = pyodide.toPy({in1, in2})
        const result = pyodide.runPython("in1 + in2", { globals: context })
        return result
    }
</script>

<h1>Pythonで足し算</h1>

<div>
    <input type="number" bind:value={in1}/> + <input type="number" bind:value={in2}/> = {#await res} caluculating... {:then result} {result} {:catch name} Error... {name} {/await}
</div>
