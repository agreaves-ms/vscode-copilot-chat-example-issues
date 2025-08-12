# vscode-copilot-chat-example-issues

Example issues when using VS Code GitHub Copilot Chat.

---

### gpt-5/apply-patch-markdown-20250812

[gpt-5/apply-patch-markdown-20250812](./examples/gpt-5/apply-patch-markdown-20250812)

Contrived example with multiple same sections and asking gpt-5 to make a change to a specific section.

**Demonstrates**:

- Issues with the apply_patch tool and missing formatting and unique context.

**Logs**:

- [Chat Conversation](results/gpt-5/apply-patch-markdown-20250812/20250813-1350/000-chat-conversation.md)
- [File Being Worked On](./results/gpt-5/apply-patch-markdown-20250812/20250813-1350/research-document.md)
- [Result](./results/gpt-5/apply-patch-markdown-20250812/20250813-1350/research-document.result.md)
- [**Captured Logs README**](./results/gpt-5/apply-patch-markdown-20250812/20250813-1350/README.md)

---

### claude-sonnet-4/replace-string-in-file-markdown-20250812

[claude-sonnet-4/replace-string-in-file-markdown-20250812](./examples/claude-sonnet-4/replace-string-in-file-markdown-20250812)

Contrived example with read_file -> fetch -> replace_string_in_file resulting in corrupted markdown file from healStringReplace choosing the wrong target snippit to replace.

**Demonstrates**:

- Issues with claude-sonnet-4 choosing non-existant content for `"oldString"` and resulting in healStringReplace making the wrong decision for a target snippit to replace.

**Logs**:

- [Chat Conversation](./results/claude-sonnet-4/replace-string-in-file-markdown-20250812/20250813-1527/000-chat-conversation.md)
- [File Being Worked On](./results/claude-sonnet-4/replace-string-in-file-markdown-20250812/20250813-1527/research-document.md)
- [Result](./results/claude-sonnet-4/replace-string-in-file-markdown-20250812/20250813-1527/research-document.result.md)
- [**Captured Logs README**](./results/claude-sonnet-4/replace-string-in-file-markdown-20250812/20250813-1527/README.md)
