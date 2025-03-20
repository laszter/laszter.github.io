<script lang="ts">
    import { Link } from "svelte-routing";
    import { PDFDocument } from "@cantoo/pdf-lib";

    let filePdf: File | null = null;
    let password: string = "";

    const onFileChange = (e: Event) => {
        const file = (e.target as HTMLInputElement).files?.[0];
        filePdf = file || null;
    };

    const unlockPdfFile = async () => {
        if (filePdf) {
            const arrayBuffer = await filePdf.arrayBuffer();

            try {
                const newPdfDoc = await PDFDocument.load(arrayBuffer, {
                    password,
                });

                // Serialize the new PDF
                const newPdfBytes = await newPdfDoc.save();

                // Convert new PDF bytes to Blob
                const blob = new Blob([newPdfBytes], {
                    type: "application/pdf",
                });

                // Download the unlocked copy
                const url = URL.createObjectURL(blob);
                const a = document.createElement("a");
                a.href = url;
                a.download = filePdf.name.replace(".pdf", "-unlock.pdf");
                a.click();
                URL.revokeObjectURL(url);
            } catch (error: any) {
                if (error.name === "PasswordException") {
                    alert("Incorrect password. Please try again.");
                } else {
                    console.error("Error unlocking PDF:", error);
                }
            }
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
    <h1>🔓 Unlock PDF</h1>
    <div>
        <div
            class="w-full border-2 border-dashed border-neutral-300 rounded-lg p-4 my-8"
        >
            <label for="pdf-file" class="cursor-pointer">
                <input
                    type="file"
                    id="pdf-file"
                    class="hidden"
                    accept=".pdf"
                    on:change={onFileChange}
                />
                <span class="text-neutral-500"
                    >{filePdf
                        ? filePdf.name
                        : "Click to upload a PDF file"}</span
                >
            </label>
        </div>
        <input
            type="password"
            id="pdf-password"
            class="w-full input-text p-4 my-8 border-2 border-neutral-300 rounded-lg"
            placeholder="Password"
            bind:value={password}
        />
        <button
            on:click={unlockPdfFile}
            class="cursor-pointer bg-neutral-900 hover:bg-neutral-700 py-4 px-4 text-white font-semibold rounded-lg transition-all w-full"
            >Unlock</button
        >
    </div>
</main>
