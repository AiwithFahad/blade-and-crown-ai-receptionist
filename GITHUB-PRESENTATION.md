# GitHub Presentation Guide — Blade & Crown

## Recommended repository name

`blade-and-crown-ai-receptionist`

## Repository description (one line)

> AI voice receptionist & booking automation for a barbershop — Retell AI ("Maya") + n8n + Supabase + Google Calendar, documented end-to-end from the live demo.

## Topics / tags

```
n8n  retell-ai  voice-ai  ai-receptionist  ai-agent  supabase
google-calendar  workflow-automation  appointment-booking  barbershop
nextjs  gmail-automation  webhook  portfolio
```

## Suggested commit messages

```
docs: add README with architecture overview and demo screenshots
docs: document all four n8n workflows node-by-node
docs: add project architecture with Mermaid data-flow diagram
docs: add setup/reconstruction guide (verified vs suggested)
docs: add limitations & disclosure (video-based documentation)
assets: add screenshots extracted from the demo video
docs: add demo video link placeholder with timestamp index
```

## Upload instructions

1. Create a new **public** repository on GitHub named `blade-and-crown-ai-receptionist` (do not initialize it with a README — this pack contains one).
2. From the unzipped folder:
   ```bash
   cd blade-and-crown-ai-receptionist
   git init
   git add .
   git commit -m "docs: Blade & Crown AI receptionist — full portfolio documentation"
   git branch -M main
   git remote add origin https://github.com/AiwithFahad/blade-and-crown-ai-receptionist.git
   git push -u origin main
   ```
   *Alternative (no terminal):* create the empty repo on github.com, click **uploading an existing file**, drag in all files and folders, and commit.
3. In the repo's **About** panel (gear icon): paste the one-line description above and add the topics listed above.
4. Replace the placeholder in `demo/VIDEO-LINK.md` with the real video URL once the demo is hosted, and add that URL to the About panel's website field.
5. Pin the repository on the GitHub profile if it is among your strongest n8n/voice-AI builds.

## Presentation advice

- **Lead with the demo.** Recruiters and clients will watch 30 seconds of Maya booking a real appointment before they read anything — put the video link at the very top of the README once hosted.
- **Keep the honesty framing.** The "documentation from the demo video" disclosure (PROJECT-LIMITATIONS.md) builds credibility; do not remove it. When you can, strengthen the repo by adding the real workflow JSON exports and a sanitized Supabase schema — then update the limitations file accordingly.
- **Use the execution screenshot.** The "REAL EXECUTION · SUCCEEDED" frame is the strongest proof asset in the pack; it is already in the README.
- **Never commit credentials.** The real build uses a webhook secret token, a Supabase shared secret, and Google/Gmail OAuth credentials — none of these belong in any public repo, screenshot, or video. Re-check frames before publishing the video anywhere.
- **Refresh the numbers.** If the system evolves (more tools for Maya, SMS notifications, payments), update README's feature list and the workflow doc in the same commit so the docs never drift from the build.
