<script>
    import { loadPyodideInstance } from "$lib/loadPyodide";
    import { text } from "@sveltejs/kit";
    import { onMount } from "svelte";

    let errList = $state([])
    let outList = $state([])
    let editordiv
    let pyEditor
    let moduleName = $state()

    onMount(async() => {
        const { editor, KeyCode, KeyMod } = await import('monaco-editor/esm/vs/editor/editor.main.js')
        pyodide = await loadPyodideInstance()
        await pyodide.loadPackage("micropip")
        pyodide.setStdout({batched: (str) => outList.push(str)})
        pyodide.setStderr({batched: (str) => outList.push(str)})
        pyEditor = editor.create(editordiv, {
            value: "",
            language: "python",
            automaticLayout: true,
            theme: "vs-dark"
        })
        pyEditor.addCommand(KeyMod.CtrlCmd | KeyCode.Enter, exeScript)
    })

    let installed_packages;

    async function exeScript() {
        const userPyScript = pyEditor.getValue()
        pyodide = await loadPyodideInstance()
        try {
            if (userPyScript === "") {
                const micropip = pyodide.pyimport("micropip")
                installed_packages = micropip.list()
                console.log(micropip)
                let output = ""
                for (const [package_name, package_version] of Object.entries(installed_packages.toJs())) {
                    output += package_name + "==" + package_version + "\\n"
                }
                outList.push(output)
            } else {
                pyodide.runPython(userPyScript)
            }
        }
        catch(e) {
            errList.push(e)
        }
    }

    async function get_installed_packages() {
        pyodide = await loadPyodideInstance()
        try {
            const installed_package = pyodide.runPython(`
import micropip

installed_package = micropip.list()

print(installed_package)
`)
            outList.push(installed_package)
        } catch (e) {
            errList.push(e)
        }
    }

    async function pipinstall(pythonModule) {
        pyodide = await loadPyodideInstance()
        const micropip = pyodide.pyimport("micropip")
        return await micropip.install(pythonModule)
    }
</script>

<h1>ユーザスクリプト実行</h1>

<input type="text" bind:value={moduleName}>

<button onclick={() => (pipinstall(moduleName))}>pip install</button>

<button onclick={get_installed_packages}>List installed packages</button>

<h2>Pythonスクリプトエディタ</h2>

<div class="editor" bind:this={editordiv}></div>

<button onclick={exeScript}>Run! (ctrl + Enter)</button>

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