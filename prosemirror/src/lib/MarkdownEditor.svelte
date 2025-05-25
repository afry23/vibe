<script lang="ts">
    import { onMount, onDestroy } from "svelte";
    import { EditorView } from "prosemirror-view";
    import { EditorState } from "prosemirror-state";
    import {
        schema,
        defaultMarkdownParser,
        defaultMarkdownSerializer,
    } from "prosemirror-markdown";
    import { exampleSetup } from "prosemirror-example-setup";

    // Props
    export let initialContent: string =
        "# Hello World\n\nThis is some **bold** text and *italic* text.\n\n- List item 1\n- List item 2\n- List item 3\n\n> This is a blockquote";

    // Component state
    let editorContainer: HTMLDivElement;
    let viewMode: "markdown" | "prosemirror" = "prosemirror";
    let currentView: MarkdownView | ProseMirrorView | null = null;
    let currentContent: string = initialContent;

    // Abstract interface for both views
    interface EditorInterface {
        get content(): string;
        focus(): void;
        destroy(): void;
    }

    // Textarea-based markdown view
    class MarkdownView implements EditorInterface {
        private textarea: HTMLTextAreaElement;
        private container: HTMLElement;

        constructor(target: HTMLElement, content: string) {
            this.container = target;
            this.textarea = document.createElement("textarea");
            this.textarea.value = content;
            this.textarea.className = "markdown-textarea";

            // Clear target and add textarea
            this.container.innerHTML = "";
            this.container.appendChild(this.textarea);
        }

        get content(): string {
            return this.textarea.value;
        }

        focus(): void {
            this.textarea.focus();
        }

        destroy(): void {
            if (this.container) {
                this.container.innerHTML = "";
            }
        }
    }

    // ProseMirror-based view
    class ProseMirrorView implements EditorInterface {
        private view: EditorView;
        private container: HTMLElement;

        constructor(target: HTMLElement, content: string) {
            this.container = target;

            // Clear target completely
            this.container.innerHTML = "";

            this.view = new EditorView(this.container, {
                state: EditorState.create({
                    doc: defaultMarkdownParser.parse(content),
                    plugins: exampleSetup({ schema }),
                }),
            });
        }

        get content(): string {
            return defaultMarkdownSerializer.serialize(this.view.state.doc);
        }

        focus(): void {
            this.view.focus();
        }

        destroy(): void {
            if (this.view) {
                this.view.destroy();
            }
            if (this.container) {
                this.container.innerHTML = "";
            }
        }
    }

    // Switch between view modes
    function switchView(newMode: "markdown" | "prosemirror") {
        if (viewMode === newMode && currentView) return;

        // Save current content
        if (currentView) {
            currentContent = currentView.content;
            currentView.destroy();
            currentView = null;
        }

        // Wait a tick for cleanup, then create new view
        setTimeout(() => {
            const ViewClass =
                newMode === "markdown" ? MarkdownView : ProseMirrorView;
            currentView = new ViewClass(editorContainer, currentContent);
            viewMode = newMode;

            // Focus after another tick
            setTimeout(() => {
                if (currentView) {
                    currentView.focus();
                }
            }, 10);
        }, 10);
    }

    // Initialize the editor when component mounts
    onMount(() => {
        switchView("prosemirror");
    });

    // Cleanup when component is destroyed
    onDestroy(() => {
        if (currentView) {
            currentView.destroy();
            currentView = null;
        }
    });

    // Export function to get current content
    export function getContent(): string {
        return currentView ? currentView.content : currentContent;
    }

    // Export function to set content
    export function setContent(content: string): void {
        currentContent = content;
        if (currentView) {
            const currentMode = viewMode;
            currentView.destroy();
            currentView = null;

            setTimeout(() => {
                const ViewClass =
                    currentMode === "markdown" ? MarkdownView : ProseMirrorView;
                currentView = new ViewClass(editorContainer, content);

                setTimeout(() => {
                    if (currentView) {
                        currentView.focus();
                    }
                }, 10);
            }, 10);
        }
    }

    // Handle radio button changes
    function handleModeChange(event: Event) {
        const target = event.target as HTMLInputElement;
        if (target.checked) {
            switchView(target.value as "markdown" | "prosemirror");
        }
    }
</script>

<div class="editor-wrapper">
    <div class="mode-selector">
        <label class="radio-label">
            <input
                type="radio"
                name="editor-mode"
                value="prosemirror"
                checked={viewMode === "prosemirror"}
                on:change={handleModeChange}
            />
            Rich Editor (ProseMirror)
        </label>

        <label class="radio-label">
            <input
                type="radio"
                name="editor-mode"
                value="markdown"
                checked={viewMode === "markdown"}
                on:change={handleModeChange}
            />
            Markdown Source
        </label>
    </div>

    <div class="editor-container" bind:this={editorContainer}></div>

    <div class="mode-info">
        Current mode: <strong
            >{viewMode === "prosemirror"
                ? "Rich Editor"
                : "Markdown Source"}</strong
        >
    </div>
</div>

<style>
    .editor-wrapper {
        max-width: 800px;
        margin: 0 auto;
        font-family:
            -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    }

    .mode-selector {
        display: flex;
        gap: 20px;
        margin-bottom: 16px;
        padding: 12px 16px;
        background: #f8f9fa;
        border-radius: 8px;
        border: 1px solid #e9ecef;
    }

    .radio-label {
        display: flex;
        align-items: center;
        gap: 8px;
        cursor: pointer;
        font-size: 14px;
        font-weight: 500;
        color: #495057;
    }

    .radio-label input[type="radio"] {
        margin: 0;
        cursor: pointer;
        accent-color: #007acc;
    }

    .editor-container {
        min-height: 400px;
        border-radius: 6px;
        position: relative;
    }

    .mode-info {
        margin-top: 12px;
        padding: 8px 12px;
        background: #e3f2fd;
        border: 1px solid #bbdefb;
        border-radius: 4px;
        font-size: 13px;
        color: #1565c0;
    }

    /* Markdown Textarea Styling */
    :global(.markdown-textarea) {
        width: 100%;
        height: 400px;
        padding: 16px;
        border: 2px solid #28a745;
        border-radius: 6px;
        font-family: "Monaco", "Menlo", "Ubuntu Mono", monospace;
        font-size: 14px;
        line-height: 1.6;
        background: #f8fff9;
        color: #333;
        resize: vertical;
        outline: none;
        box-sizing: border-box;
    }

    :global(.markdown-textarea:focus) {
        border-color: #20c997;
        background: #f0fff4;
        box-shadow: 0 0 0 3px rgba(40, 167, 69, 0.25);
    }

    :global(.markdown-textarea::placeholder) {
        color: #6c757d;
        font-style: italic;
    }

    /* ProseMirror Menu Bar Styling */
    :global(.ProseMirror-menubar) {
        border: 2px solid #007acc;
        border-bottom: 1px solid #007acc;
        border-top-left-radius: 6px;
        border-top-right-radius: 6px;
        background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
        padding: 8px;
        display: flex;
        flex-wrap: wrap;
        gap: 4px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        min-height: 44px;
        max-height: 44px;
        height: 44px;
        overflow: hidden;
        align-items: center;
        box-sizing: border-box;
    }

    :global(.ProseMirror-menu) {
        display: flex;
        flex-wrap: nowrap;
        gap: 2px;
        align-items: center;
        height: 28px;
        overflow: hidden;
    }

    :global(.ProseMirror-menuitem) {
        background: white;
        border: 1px solid #dee2e6;
        border-radius: 4px;
        cursor: pointer;
        padding: 4px 6px;
        font-size: 14px;
        font-weight: 600;
        color: #495057;
        min-width: 24px;
        max-width: 32px;
        height: 24px;
        text-align: center;
        display: inline-flex;
        align-items: center;
        justify-content: center;
        transition: all 0.2s ease;
        flex-shrink: 0;
        box-sizing: border-box;
        line-height: 1;
    }

    :global(.ProseMirror-menuitem:hover) {
        background: #e9ecef;
        border-color: #adb5bd;
        transform: translateY(-1px);
    }

    :global(.ProseMirror-menuitem.ProseMirror-menu-active) {
        background: #007acc;
        color: white;
        border-color: #0056b3;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    }

    :global(.ProseMirror-menuitem.ProseMirror-menu-disabled) {
        color: #adb5bd;
        cursor: not-allowed;
        opacity: 0.6;
    }

    :global(.ProseMirror-menuitem.ProseMirror-menu-disabled:hover) {
        background: white;
        border-color: #dee2e6;
        transform: none;
    }

    :global(.ProseMirror-menuseparator) {
        border-left: 1px solid #adb5bd;
        margin: 0 4px;
        height: 16px;
        flex-shrink: 0;
    }

    /* ProseMirror Editor Styling */
    :global(.ProseMirror) {
        padding: 16px;
        border: 2px solid #007acc;
        border-top: none;
        border-bottom-left-radius: 6px;
        border-bottom-right-radius: 6px;
        min-height: 380px;
        outline: none;
        font-size: 14px;
        line-height: 1.6;
        background: #fafbfc;
    }

    :global(.ProseMirror:focus) {
        background: white;
        box-shadow: 0 0 0 3px rgba(0, 122, 204, 0.25);
    }

    :global(.ProseMirror h1) {
        font-size: 28px;
        font-weight: bold;
        margin: 20px 0 12px 0;
        color: #212529;
        border-bottom: 2px solid #e9ecef;
        padding-bottom: 8px;
    }

    :global(.ProseMirror h2) {
        font-size: 22px;
        font-weight: bold;
        margin: 18px 0 10px 0;
        color: #343a40;
    }

    :global(.ProseMirror h3) {
        font-size: 18px;
        font-weight: bold;
        margin: 16px 0 8px 0;
        color: #495057;
    }

    :global(.ProseMirror p) {
        margin: 10px 0;
        color: #212529;
    }

    :global(.ProseMirror ul, .ProseMirror ol) {
        margin: 10px 0;
        padding-left: 28px;
    }

    :global(.ProseMirror li) {
        margin: 4px 0;
    }

    :global(.ProseMirror blockquote) {
        border-left: 4px solid #007acc;
        padding-left: 16px;
        margin: 16px 0;
        font-style: italic;
        color: #6c757d;
        background: #f8f9fa;
        padding: 12px 16px;
        border-radius: 0 4px 4px 0;
    }

    :global(.ProseMirror code) {
        background-color: #f1f3f4;
        padding: 3px 6px;
        border-radius: 4px;
        font-family: "Monaco", "Menlo", "Ubuntu Mono", monospace;
        font-size: 13px;
        color: #d63384;
        border: 1px solid #e9ecef;
    }

    :global(.ProseMirror pre) {
        background-color: #f8f9fa;
        padding: 16px;
        border-radius: 6px;
        overflow-x: auto;
        margin: 16px 0;
        border: 1px solid #e9ecef;
    }

    :global(.ProseMirror pre code) {
        background: none;
        padding: 0;
        border: none;
        color: #495057;
    }

    /* Dropdown menu styling */
    :global(.ProseMirror-menu-dropdown) {
        position: relative;
        display: inline-block;
    }

    :global(.ProseMirror-menu-dropdown-menu) {
        position: absolute;
        top: 100%;
        left: 0;
        background: white;
        border: 1px solid #dee2e6;
        border-radius: 4px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        z-index: 1000;
        min-width: 140px;
    }

    :global(.ProseMirror-menu-dropdown-item) {
        display: block;
        padding: 8px 12px;
        cursor: pointer;
        border: none;
        background: none;
        text-align: left;
        width: 100%;
        font-size: 14px;
        color: #495057;
    }

    :global(.ProseMirror-menu-dropdown-item:hover) {
        background: #f8f9fa;
    }
</style>
