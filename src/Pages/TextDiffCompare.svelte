<script lang="ts">
    import { Link } from "svelte-routing";

    type DiffSegment = {
        value: string;
        type: "same" | "added" | "removed";
    };

    type DiffRow = {
        leftLineNumber: number | null;
        rightLineNumber: number | null;
        kind: "same" | "added" | "removed" | "changed";
        leftSegments: DiffSegment[];
        rightSegments: DiffSegment[];
    };

    let leftText = $state("");
    let rightText = $state("");
    let ignoreCase = $state(false);
    let ignoreWhitespace = $state(false);
    let syncScroll = $state(true);

    let leftInputEl: HTMLTextAreaElement | undefined;
    let rightInputEl: HTMLTextAreaElement | undefined;
    let leftResultEl: HTMLDivElement | undefined;
    let rightResultEl: HTMLDivElement | undefined;

    const emptySegments = (): DiffSegment[] => [{ value: "", type: "same" }];

    const tokenize = (value: string): string[] =>
        value.match(/\s+|[A-Za-z0-9_]+|[^A-Za-z0-9_\s]+/g) ?? [];

    const normalizeValue = (value: string) => {
        let normalized = value.replace(/\r/g, "");

        if (ignoreWhitespace) {
            normalized = normalized.replace(/\s+/g, "");
        }

        if (ignoreCase) {
            normalized = normalized.toLowerCase();
        }

        return normalized;
    };

    const buildLcsMatrix = <T,>(
        left: T[],
        right: T[],
        isEqual: (a: T, b: T) => boolean,
    ) => {
        const matrix = Array.from({ length: left.length + 1 }, () =>
            Array(right.length + 1).fill(0),
        );

        for (let leftIndex = left.length - 1; leftIndex >= 0; leftIndex -= 1) {
            for (
                let rightIndex = right.length - 1;
                rightIndex >= 0;
                rightIndex -= 1
            ) {
                matrix[leftIndex][rightIndex] = isEqual(
                    left[leftIndex],
                    right[rightIndex],
                )
                    ? matrix[leftIndex + 1][rightIndex + 1] + 1
                    : Math.max(
                          matrix[leftIndex + 1][rightIndex],
                          matrix[leftIndex][rightIndex + 1],
                      );
            }
        }

        return matrix;
    };

    const diffTokens = (leftValue: string, rightValue: string) => {
        const leftTokens = tokenize(leftValue);
        const rightTokens = tokenize(rightValue);
        const matrix = buildLcsMatrix(
            leftTokens,
            rightTokens,
            (leftToken, rightToken) =>
                normalizeValue(leftToken) === normalizeValue(rightToken),
        );

        const leftSegments: DiffSegment[] = [];
        const rightSegments: DiffSegment[] = [];

        let leftIndex = 0;
        let rightIndex = 0;

        while (leftIndex < leftTokens.length && rightIndex < rightTokens.length) {
            if (
                normalizeValue(leftTokens[leftIndex]) ===
                normalizeValue(rightTokens[rightIndex])
            ) {
                leftSegments.push({
                    value: leftTokens[leftIndex],
                    type: "same",
                });
                rightSegments.push({
                    value: rightTokens[rightIndex],
                    type: "same",
                });
                leftIndex += 1;
                rightIndex += 1;
                continue;
            }

            if (matrix[leftIndex + 1][rightIndex] >= matrix[leftIndex][rightIndex + 1]) {
                leftSegments.push({
                    value: leftTokens[leftIndex],
                    type: "removed",
                });
                leftIndex += 1;
                continue;
            }

            rightSegments.push({
                value: rightTokens[rightIndex],
                type: "added",
            });
            rightIndex += 1;
        }

        while (leftIndex < leftTokens.length) {
            leftSegments.push({
                value: leftTokens[leftIndex],
                type: "removed",
            });
            leftIndex += 1;
        }

        while (rightIndex < rightTokens.length) {
            rightSegments.push({
                value: rightTokens[rightIndex],
                type: "added",
            });
            rightIndex += 1;
        }

        return {
            leftSegments: leftSegments.length > 0 ? leftSegments : emptySegments(),
            rightSegments: rightSegments.length > 0 ? rightSegments : emptySegments(),
        };
    };

    const createChangedRow = (
        leftLine: string,
        rightLine: string,
        leftLineNumber: number | null,
        rightLineNumber: number | null,
    ): DiffRow => {
        const tokenDiff = diffTokens(leftLine, rightLine);

        return {
            leftLineNumber,
            rightLineNumber,
            kind: "changed",
            leftSegments: tokenDiff.leftSegments,
            rightSegments: tokenDiff.rightSegments,
        };
    };

    const getLines = (value: string) => value.replace(/\r/g, "").split("\n");

    const buildRows = (leftValue: string, rightValue: string): DiffRow[] => {
        const leftLines = getLines(leftValue);
        const rightLines = getLines(rightValue);

        const leftHasContent = leftLines.length > 1 || leftLines[0] !== "";
        const rightHasContent = rightLines.length > 1 || rightLines[0] !== "";

        if (!leftHasContent && !rightHasContent) {
            return [];
        }

        const matrix = buildLcsMatrix(
            leftLines,
            rightLines,
            (leftLine, rightLine) =>
                normalizeValue(leftLine) === normalizeValue(rightLine),
        );

        const commonPairs: Array<{ leftIndex: number; rightIndex: number }> = [];
        let leftIndex = 0;
        let rightIndex = 0;

        while (leftIndex < leftLines.length && rightIndex < rightLines.length) {
            if (
                normalizeValue(leftLines[leftIndex]) ===
                normalizeValue(rightLines[rightIndex])
            ) {
                commonPairs.push({ leftIndex, rightIndex });
                leftIndex += 1;
                rightIndex += 1;
                continue;
            }

            if (matrix[leftIndex + 1][rightIndex] >= matrix[leftIndex][rightIndex + 1]) {
                leftIndex += 1;
            } else {
                rightIndex += 1;
            }
        }

        const rows: DiffRow[] = [];
        let previousLeftIndex = 0;
        let previousRightIndex = 0;

        const pushPendingRows = (nextLeftIndex: number, nextRightIndex: number) => {
            const leftChunk = leftLines.slice(previousLeftIndex, nextLeftIndex);
            const rightChunk = rightLines.slice(previousRightIndex, nextRightIndex);
            const rowCount = Math.max(leftChunk.length, rightChunk.length);

            for (let chunkIndex = 0; chunkIndex < rowCount; chunkIndex += 1) {
                const leftLine = leftChunk[chunkIndex];
                const rightLine = rightChunk[chunkIndex];
                const currentLeftLineNumber =
                    leftLine !== undefined ? previousLeftIndex + chunkIndex + 1 : null;
                const currentRightLineNumber =
                    rightLine !== undefined ? previousRightIndex + chunkIndex + 1 : null;

                if (leftLine !== undefined && rightLine !== undefined) {
                    rows.push(
                        createChangedRow(
                            leftLine,
                            rightLine,
                            currentLeftLineNumber,
                            currentRightLineNumber,
                        ),
                    );
                    continue;
                }

                if (leftLine !== undefined) {
                    rows.push({
                        leftLineNumber: currentLeftLineNumber,
                        rightLineNumber: null,
                        kind: "removed",
                        leftSegments: [{ value: leftLine, type: "removed" }],
                        rightSegments: emptySegments(),
                    });
                    continue;
                }

                rows.push({
                    leftLineNumber: null,
                    rightLineNumber: currentRightLineNumber,
                    kind: "added",
                    leftSegments: emptySegments(),
                    rightSegments: [{ value: rightLine ?? "", type: "added" }],
                });
            }
        };

        for (const pair of commonPairs) {
            pushPendingRows(pair.leftIndex, pair.rightIndex);

            rows.push({
                leftLineNumber: pair.leftIndex + 1,
                rightLineNumber: pair.rightIndex + 1,
                kind: "same",
                leftSegments: [{ value: leftLines[pair.leftIndex], type: "same" }],
                rightSegments: [{ value: rightLines[pair.rightIndex], type: "same" }],
            });

            previousLeftIndex = pair.leftIndex + 1;
            previousRightIndex = pair.rightIndex + 1;
        }

        pushPendingRows(leftLines.length, rightLines.length);

        return rows;
    };

    const rows = $derived(buildRows(leftText, rightText));
    const changedCount = $derived(
        rows.filter((row) => row.kind === "changed").length,
    );
    const addedCount = $derived(rows.filter((row) => row.kind === "added").length);
    const removedCount = $derived(
        rows.filter((row) => row.kind === "removed").length,
    );
    const leftLineCount = $derived(
        leftText === "" ? 0 : getLines(leftText).length,
    );
    const rightLineCount = $derived(
        rightText === "" ? 0 : getLines(rightText).length,
    );

    const syncVerticalScroll = (
        source: HTMLElement,
        target: HTMLElement | undefined,
    ) => {
        if (!syncScroll || !target) return;

        target.scrollTop = source.scrollTop;
        target.scrollLeft = source.scrollLeft;
    };

    const handleInputScroll = (
        event: Event,
        target: HTMLTextAreaElement | undefined,
    ) => {
        if (!event.currentTarget) return;
        syncVerticalScroll(event.currentTarget as HTMLTextAreaElement, target);
    };

    const handleResultScroll = (event: Event, target: HTMLDivElement | undefined) => {
        if (!event.currentTarget) return;
        syncVerticalScroll(event.currentTarget as HTMLDivElement, target);
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

    <h1>🧩 Text Diff Compare</h1>
    <p class="tool-subtitle">
        Compare two texts side by side and inspect line-by-line or word-level changes.
    </p>

    <div class="option-box">
        <div class="option-item summary-item">
            <span>Changed</span>
            <strong>{changedCount}</strong>
            <span>Added</span>
            <strong>{addedCount}</strong>
            <span>Removed</span>
            <strong>{removedCount}</strong>
        </div>
        <div class="option-item">
            Ignore case
            <label class="switch">
                <input type="checkbox" bind:checked={ignoreCase} />
                <div class="slider"></div>
            </label>
            Ignore whitespace
            <label class="switch">
                <input type="checkbox" bind:checked={ignoreWhitespace} />
                <div class="slider"></div>
            </label>
            Sync scroll
            <label class="switch">
                <input type="checkbox" bind:checked={syncScroll} />
                <div class="slider"></div>
            </label>
        </div>
    </div>

    <section class="editor-section">
        <div class="panel">
            <div class="panel-header">
                <span>Original</span>
                <span>{leftLineCount} lines</span>
            </div>
            <textarea
                bind:this={leftInputEl}
                bind:value={leftText}
                title="Original text"
                class="input-text"
                spellcheck="false"
                placeholder="Paste the original text here..."
                onscroll={(event) => handleInputScroll(event, rightInputEl)}
            ></textarea>
        </div>

        <div class="panel">
            <div class="panel-header">
                <span>Modified</span>
                <span>{rightLineCount} lines</span>
            </div>
            <textarea
                bind:this={rightInputEl}
                bind:value={rightText}
                title="Modified text"
                class="input-text"
                spellcheck="false"
                placeholder="Paste the updated text here..."
                onscroll={(event) => handleInputScroll(event, leftInputEl)}
            ></textarea>
        </div>
    </section>

    <section class="diff-section">
        <div class="panel">
            <div class="panel-header">
                <span>Original diff</span>
                <span>{rows.length} rows</span>
            </div>
            <div
                bind:this={leftResultEl}
                class="diff-output"
                onscroll={(event) => handleResultScroll(event, rightResultEl)}
            >
                {#if rows.length === 0}
                    <div class="empty-state">
                        Add text to both editors to start comparing.
                    </div>
                {:else}
                    {#each rows as row}
                        <div class={`diff-line ${row.kind}`}>
                            <div class="line-number">
                                {row.leftLineNumber ?? ""}
                            </div>
                            <pre class="line-content">{#each row.leftSegments as segment}<span class={`segment ${segment.type}`}>{segment.value}</span>{/each}</pre>
                        </div>
                    {/each}
                {/if}
            </div>
        </div>

        <div class="panel">
            <div class="panel-header">
                <span>Modified diff</span>
                <span>{rows.length} rows</span>
            </div>
            <div
                bind:this={rightResultEl}
                class="diff-output"
                onscroll={(event) => handleResultScroll(event, leftResultEl)}
            >
                {#if rows.length === 0}
                    <div class="empty-state">
                        Added, removed, and changed lines will appear here.
                    </div>
                {:else}
                    {#each rows as row}
                        <div class={`diff-line ${row.kind}`}>
                            <div class="line-number">
                                {row.rightLineNumber ?? ""}
                            </div>
                            <pre class="line-content">{#each row.rightSegments as segment}<span class={`segment ${segment.type}`}>{segment.value}</span>{/each}</pre>
                        </div>
                    {/each}
                {/if}
            </div>
        </div>
    </section>
</main>

<style>
    .tool-subtitle {
        margin: 12px auto 0;
        max-width: 760px;
        color: #b4b4b4;
    }

    .option-box {
        display: flex;
        flex-wrap: wrap;
        gap: 12px;
        justify-content: center;
        align-items: stretch;
        margin: 24px auto;
        width: 100%;
    }

    .option-item {
        display: flex;
        gap: 12px;
        justify-content: center;
        align-items: center;
        padding: 14px 18px;
        background-color: #3f3f3f;
        border-radius: 10px;
        color: #fff;
        flex: 1 1 320px;
    }

    .summary-item {
        justify-content: flex-start;
        flex-wrap: wrap;
    }

    .summary-item strong {
        min-width: 30px;
        color: #f5f5f5;
    }

    .editor-section,
    .diff-section {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 16px;
        width: 100%;
        margin-bottom: 20px;
    }

    .panel {
        min-width: 0;
    }

    .panel-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 10px 14px;
        background: #2e2e2e;
        border-radius: 12px 12px 0 0;
        color: #d4d4d4;
        font-size: 14px;
    }

    .input-text {
        width: 100%;
        height: 280px;
        box-sizing: border-box;
        resize: vertical;
        font-family: "SFMono-Regular", "Consolas", monospace;
        font-size: 14px;
        line-height: 1.5;
        background-color: #171717;
        color: #fff;
        border: 1px solid #333;
        border-top: 0;
        border-radius: 0 0 12px 12px;
        padding: 16px;
        white-space: pre;
    }

    .diff-output {
        height: 420px;
        overflow: auto;
        background: #171717;
        border: 1px solid #333;
        border-top: 0;
        border-radius: 0 0 12px 12px;
        text-align: left;
    }

    .diff-line {
        display: grid;
        grid-template-columns: 56px minmax(0, 1fr);
        min-height: 28px;
        border-bottom: 1px solid rgba(255, 255, 255, 0.05);
    }

    .diff-line.same {
        background: transparent;
    }

    .diff-line.added {
        background: rgba(34, 197, 94, 0.14);
    }

    .diff-line.removed {
        background: rgba(239, 68, 68, 0.14);
    }

    .diff-line.changed {
        background: rgba(250, 204, 21, 0.12);
    }

    .line-number {
        padding: 6px 10px;
        text-align: right;
        color: #888;
        background: rgba(255, 255, 255, 0.03);
        border-right: 1px solid rgba(255, 255, 255, 0.05);
        user-select: none;
    }

    .line-content {
        margin: 0;
        padding: 6px 12px;
        white-space: pre-wrap;
        word-break: break-word;
        font-family: "SFMono-Regular", "Consolas", monospace;
        color: #f4f4f4;
    }

    .segment.added {
        background: rgba(34, 197, 94, 0.3);
        color: #d1fae5;
        border-radius: 4px;
    }

    .segment.removed {
        background: rgba(239, 68, 68, 0.3);
        color: #fee2e2;
        border-radius: 4px;
    }

    .empty-state {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100%;
        min-height: 240px;
        color: #8f8f8f;
        padding: 24px;
        text-align: center;
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

    @media (max-width: 900px) {
        .editor-section,
        .diff-section {
            grid-template-columns: 1fr;
        }

        .input-text {
            height: 220px;
        }

        .diff-output {
            height: 320px;
        }
    }
</style>
