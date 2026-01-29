# Agent-TUI Eval Creation Flow

<prompt>
Using agent-tui, launch Claude Code in a new random tmp directory. Inside the Claude Code session:
1. Create a file called data.txt with content 'Test Content 99999'
2. Duplicate it to data_copy.txt
3. Verify the copy matches using diff
4. Run /evals create file-copy to create an eval of this process
5. Move the eval file into an evals/ subdirectory

Approve all permission prompts. Kill the session when done.
</prompt>

<expectation>
The tmp directory contains:
- data.txt and data_copy.txt with identical content
- evals/file-copy.eval.md (or similar name) exists with valid <prompt> and <expectation> tags
</expectation>
