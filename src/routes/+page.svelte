<script>
    const pages = [
        {
            link: "/pyodide_test/calc_add",
            text: "Pythonで足し算"
        },
        {
            link: "/pyodide_test/user_script_exe",
            text: "ユーザスクリプト実行"
        }
    ]
</script>

<h1>Pyodideで遊んでみた</h1>

<h2>作ったもの一覧</h2>

{#each pages as page}
<a href={page.link}>{page.text}</a>
<br>
{/each}
