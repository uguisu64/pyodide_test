<script>
    import { loadPyodideInstance } from "$lib/loadPyodide";
    import { onMount } from "svelte";
    import { editor } from "monaco-editor";

    let errList = $state([])
    let outList = $state([])
    let editordiv
    let pyEditor

    onMount(async() => {
        pyodide = await loadPyodideInstance()
        pyodide.setStdout({batched: (str) => outList.push(str)})
        pyodide.setStderr({batched: (str) => outList.push(str)})
        pyEditor = editor.create(editordiv, {
            value: "",
            language: "python",
            automaticLayout: true,
            theme: "vs-dark"
        })
    })

    async function exeScript() {
        const userPyScript = pyEditor.getValue()
        pyodide = await loadPyodideInstance()
        try {
            pyodide.runPython(userPyScript)
        }
        catch(e) {
            errList.push(e)
        }
    }
</script>

<h1>ユーザスクリプト実行</h1>

<h2>Pythonスクリプトエディタ</h2>

<div class="editor" bind:this={editordiv}></div>

<button onclick={exeScript}>Run!</button>

<h2>アウトプット</h2>

<div class="result">
    <section>
        <h3>stdout</h3>
        {#each outList as out}
            {out}
            <br>
        {/each}
    </section>
    <section>
        <h3>Error Message</h3>
        {#each errList as err}
           <p class="errMsg">{err}</p>
        {/each}
    </section>
</div>

<style>
    section {
        width: 50%;
        box-sizing: border-box;
        border: 1px solid #ddd;
    }

    .editor {
        width: auto;
        height: 50vh;
    }

    .result {
        display: flex;
        justify-content: flex-start;
    }

    button {
        font-size: large;
        background-color: rgb(119, 255, 119);
        border-color: none;
    }

    .errMsg {
        background-color: rgb(205, 116, 116);
        margin-left: 1pt;
    }
</style>