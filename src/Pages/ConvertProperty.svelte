<script lang="ts">
    import { Link } from "svelte-routing";

    let outputText: string = "";

    const onTextAreaScroll = (e: Event) => {
        const textareas = document.querySelectorAll("textarea");
        for (let j = 0; j < textareas.length; j++) {
            if (e.target) {
                textareas[j].scrollTop = (e.target as HTMLElement).scrollTop;
                textareas[j].scrollLeft = (e.target as HTMLElement).scrollLeft;
            }
        }
    };

    const convertProperty = (e: Event) => {
        let source = (e.currentTarget as HTMLTextAreaElement).value;
        source = source.replace(/private/gi, "public");
        source = source.replace(/string/gi, "string?");
        source = source.replace(/ _/gi, " ");
        source = source.replace(/;/gi, " { get; set; }");
        outputText = source;
    };
</script>

<main class="w-full inline-block">
    <div class="text-left">
        <Link to="/">
            <button
                class="cursor-pointer bg-neutral-900 hover:bg-neutral-700 py-2 px-4 text-white font-semibold rounded-lg transition-all"
                >🏠 Back</button
            >
        </Link>
    </div>
    <h1>⚒️ Property Converter</h1>
    <div class="desc-header">
        Replace&nbsp;&nbsp;
        <div class="code-box">private string _name;</div>
        &nbsp;&nbsp;to&nbsp;&nbsp;
        <div class="code-box">public string? name {"{ get; set; }"}</div>
    </div>
    <div class="flex flex-row justify-between gap-4 w-full my-8">
        <textarea
            placeholder=""
            title="source"
            id="source-text"
            class="input-text"
            spellcheck="false"
            on:input={convertProperty}
            on:scroll={onTextAreaScroll}
        ></textarea>
        <textarea
            placeholder=""
            title="output"
            id="output-text"
            class="input-text"
            spellcheck="false"
            bind:value={outputText}
            on:scroll={onTextAreaScroll}
        ></textarea>
    </div>
</main>

<style>
    .input-text {
        width: 100%;
        height: 600px;
        box-sizing: border-box;
        resize: none;
        font-family: "Roboto", sans-serif;
        font-size: 16px;
        line-height: 1.5;
        background-color: #1f1f1f;
        color: #fff;
        white-space: nowrap;
    }

    .desc-header {
        text-align: center;
        display: flex;
        justify-content: center;
        align-items: center;
        margin: 18px;
    }

    .code-box {
        color: #fff;
        background-color: #3f3f3f;
        border: 0px solid;
        border-radius: 4px;
        padding: 4px 16px;
        font-family: monospace;
        white-space: pre-wrap;
        overflow-x: auto;
        display: inline-block;
    }
</style>
