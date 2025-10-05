# Bootstrap VSCode debugging setup for Jurassic Escape

## Your task

1. **First, check if already fully configured:**

   - Check if `.vscode/tasks.json` exists AND contains a task with label `"start-http-server"`
   - Check if `.vscode/launch.json` exists AND contains both `"Launch Chrome (HTTP)"` and `"Launch Chrome (File)"` configurations
   - If BOTH files exist with ALL required items: Report "✅ VSCode debugging is already fully configured! You're all set." and stop here.

2. **Otherwise, proceed with setup:**

   **For `.vscode/tasks.json`:**

   - If file exists: Read it and check if task with label `"start-http-server"` exists
     - If missing: Add the task to the existing tasks array
     - If present: Skip (note in summary)
   - If file doesn't exist: Create it with the task configuration from README.md "Step 1: Create `.vscode/tasks.json`"

   **For `.vscode/launch.json`:**

   - If file exists: Read it and check if configurations named `"Launch Chrome (HTTP)"` and `"Launch Chrome (File)"` exist
     - Add any missing configurations to the existing configurations array
     - Skip any that already exist (note in summary)
   - If file doesn't exist: Create it with both configurations from README.md "Step 2: Create `.vscode/launch.json`"

3. **Use the EXACT JSON from the README.md file** for all tasks and configurations.

4. **After completing setup, provide a summary:**

   - What was created (new files)
   - What was added (new tasks/configurations merged into existing files)
   - What was skipped (already existed)

5. **Remind the user:**

   - Use "Launch Chrome (HTTP)" debug configuration for VSCode breakpoint support
   - Enable Debug Mode in Settings (⚙️ button) after launching to load unminified React builds for better stack traces
   - Page will auto-reload when Debug Mode is toggled ON
