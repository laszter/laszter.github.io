<script lang="ts">
    import { Link } from "svelte-routing";

    let sourceText: string = "";
    let minify: boolean = false;
    let sortKeys: boolean = false;
    let indent: string = "2";
    let tsesp: boolean = false;

    let outputText: string = "";

    const formatJSON = () => {
        console.log(minify, sortKeys, indent, tsesp);
        try {
            const jsonObj = JSON.parse(sourceText);
            const space = minify ? 0 : parseInt(indent);

            const replacer = sortKeys
                ? (key: string, value: any) => {
                      if (Array.isArray(value)) {
                          return value.sort();
                      } else if (typeof value === "object" && value !== null) {
                          return Object.keys(value)
                              .sort()
                              .reduce((acc: any, key: string) => {
                                  acc[key] = value[key];
                                  return acc;
                              }, {});
                      }
                      return value;
                  }
                : undefined;

            let result = JSON.stringify(jsonObj, replacer, space);
            if (tsesp) {
                result = result
                    .replaceAll("/Date(", "\\/Date(")
                    .replaceAll(")/", ")\\/"); // escape date timestamp
            }

            outputText = result;
        } catch (error) {
            outputText = sourceText;
        }
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
    <h1>📄 JSON Format & Minify</h1>
    <div class="option-box">
        <div class="option-item">
            Indent
            <select
                title="Indent"
                id="indent"
                bind:value={indent}
                on:change={formatJSON}
            >
                <option value="2" selected>2</option>
                <option value="4">4</option>
            </select>
            Timestamp Escaspe
            <label class="switch">
                <input
                    type="checkbox"
                    id="tsesp"
                    name="tsesp"
                    value="tsesp"
                    bind:checked={tsesp}
                    on:change={formatJSON}
                />
                <div class="slider"></div>
            </label>
        </div>
        <div class="option-item">
            Minify
            <label class="switch">
                <input
                    type="checkbox"
                    id="minify"
                    name="minify"
                    value="minify"
                    bind:checked={minify}
                    on:change={formatJSON}
                />
                <div class="slider"></div>
            </label>
            Sort Keys
            <label class="switch">
                <input
                    type="checkbox"
                    id="sortKeys"
                    name="sortKeys"
                    value="sortKeys"
                    bind:checked={sortKeys}
                    on:change={formatJSON}
                />
                <div class="slider"></div>
            </label>
        </div>
    </div>
    <div class="flex flex-row justify-between gap-4 w-full my-8">
        <textarea
            placeholder=""
            title="source"
            id="source-text"
            class="input-text"
            spellcheck="false"
            bind:value={sourceText}
            on:input={formatJSON}
        ></textarea>
        <textarea
            placeholder=""
            title="output"
            id="output-text"
            class="input-text"
            spellcheck="false"
            bind:value={outputText}
        ></textarea>
    </div>
</main>

<style>
    .option-box {
        display: flex;
        justify-content: center;
        align-items: center;
        margin: 18px auto;
        max-width: 1200px;
    }

    .option-box .option-item {
        margin: 0 8px;
        background-color: #3f3f3f;
        border-radius: 4px;
        padding: 12px 16px;
        width: 100%;
        color: #fff;
    }

    .option-box .option-item:hover {
        background-color: #5f5f5f;
    }

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
    }

    .option-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 10px;
    }

    select {
        -webkit-appearance: none;
        -moz-appearance: none;
        appearance: none;
        width: 50px;
        height: 30px;
        background-color: #ffffff;
        border: 1px solid #d4d4d4;
        border-radius: 5px;
        padding: 5px 10px;
        font-size: 16px;
        color: #1a1a1a;
        cursor: pointer;
    }

    .switch {
        position: relative;
        display: inline-block;
        width: 45px;
        height: 25px;
        margin: 0;
        vertical-align: middle;
        cursor: pointer;
    }

    .switch input {
        display: none;
    }

    .slider {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: #d4d4d4;
        border-radius: 25px;
        transition: 0.2s;
    }

    .slider:before {
        position: absolute;
        content: "";
        height: 21px;
        width: 21px;
        left: 2px;
        bottom: 2px;
        background-color: white;
        border-radius: 50%;
        transition: 0.2s;
    }

    input:checked + .slider {
        background-color: #8f8f8f;
    }

    input:checked + .slider:before {
        transform: translateX(20px);
    }
</style>
