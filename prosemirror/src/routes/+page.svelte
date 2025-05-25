<script lang="ts">
    import MarkdownEditor from "$lib/MarkdownEditor.svelte";

    let editor: MarkdownEditor;

    function handleSave() {
        const content = editor.getContent();
        console.log("Current content:", content);
    }
</script>

<MarkdownEditor
    bind:this={editor}
    initialContent="# Your initial markdown here"
/>

<button on:click={handleSave}>Get Content</button>
