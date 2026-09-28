Trigger a GitHub Pages redeploy of the Wayne Trimm Archive site by updating the semaphore file.

Steps:
1. Write the current date and time to `.deploy-trigger` in the project root (just a single human-readable timestamp line, e.g. `Redeployed: 2026-09-28 16:45:00`)
2. Commit the file with the message: `Trigger redeploy - <timestamp>`
3. Push to origin main
4. Confirm to the user that the push succeeded and GitHub Pages will rebuild shortly