# SETUP CLAUDE FLOW AND CLAUDE MD
### Install and initialize, 
npx claude-flow@latest init

### Or using the new Ruflo name
npx ruflo@latest init

### This creates a CLAUDE.md file and sets up the agent structure inside your project.
Step 1:
Check Your Current and edit CLAUDE.md First
`vim ~/Documents/mockchat/CLAUDE.md (edit from your editor)`

Step 2: Add This At The Very Top
```
## Claude Flow Instructions

### Project Status
This project is already in active development. Do NOT start from scratch.

### Before Writing Any Code
1. Read and understand the existing codebase in mockchat-be/ and mockchat-fe/
2. Store existing conventions to memory bank:
   - Current migrations in mockchat-be/database/migrations/
   - Existing API endpoints in mockchat-be/routes/api.php
   - Existing services: GroqChatService, OpenAIChatService, OllamaChatService
   - Existing models: Conversation, Message, CustomerType
   - Vue components in mockchat-fe/src/views/
   - API wrapper in mockchat-fe/src/api/chat.ts
   - Pinia stores in mockchat-fe/src/stores/
   - Naming conventions already used in the project
3. Follow ALL existing patterns — do not introduce new conventions
4. Do not overwrite or break any existing functionality
5. New code must integrate seamlessly with what already exists

### Memory & Coordination Rules
- Initialize memory bank so all agents share the same context
- Use wiring gap matrix to ensure backend API endpoints
  match frontend API calls in mockchat-fe/src/api/chat.ts
- Every agent must read from memory before writing code
- Every agent must save their output to memory after completion

---
```
Step 4: Save and Exit

# create simple shell with all the command prompt
Step 1: Open Your .zshrc File
`vim ~/.zshrc`

Step 2: Add This At The Bottom
`alias mockchat-ai='cd ~/Documents/mockchat && claude-flow memory init && claude-flow daemon start && claude'`

Step 3: Save and Exit

Step 4: Reload Your Shell
`source ~/.zshrc`

Step 5: Test It
`Go to your folder ex: cd projects/mockchat`
`mockchat-ai`
