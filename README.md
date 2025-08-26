# Commits Breaker
Generate a commits graph breaker game from your own GitHub contributions graph.
This mini Github workflow will grab your contribution graph and generate this interesting breakout visualization for both light and dark mode:

<picture>
  <source
    media="(prefers-color-scheme: dark)"
    srcset="example/dark.svg"
  />
  <source
    media="(prefers-color-scheme: light)"
    srcset="example/light.svg"
  />
  <img alt="Breakout Game" src="example/light.svg" />
</picture>

## 1. Copy the Workflow

You don’t need to fork the whole repo. Just create a GitHub Actions workflow in your own **profile repository** by following these steps:

* Go to your profile repo (`github.com/<your-username>/<your-username>`) or create one if you havent already.
* Create a folder: `.github/workflows/`
* Inside it, add a file like `breakout.yml` with this exact content, no additional changes needed:

```yaml
name: generate breakout svg

on:
  schedule:
    - cron: "0 */24 * * *"
  workflow_dispatch:

jobs:
  generate-svg:
    permissions:
      contents: write
    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: generate SVG
        uses: cyprieng/github-breakout@v1.0.0
        with:
          github_username: ${{ github.repository_owner }}

      - name: Move generated SVGs
        run: |
          mkdir -p images
          mv output/light.svg images/breakout-light.svg
          mv output/dark.svg images/breakout-dark.svg

      - name: Configure git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Commit and push SVGs
        run: |
          git add images/breakout-light.svg images/breakout-dark.svg
          git commit -m "chore: update breakout SVGs" || echo "No changes to commit"
          git push
```

That action will automatically fetch your contribution graph and regenerate the breakout game SVGs daily.

---

## 2. Update Your Profile README

In the `README.md` of your profile repo, add this snippet:

```html
<picture>
  <source
    media="(prefers-color-scheme: dark)"
    srcset="images/breakout-dark.svg"
  />
  <source
    media="(prefers-color-scheme: light)"
    srcset="images/breakout-light.svg"
  />
  <img alt="Breakout Game" src="images/breakout-light.svg" />
</picture>
```

This way, your profile will show the light or dark breakout game depending on visitors’ GitHub theme.

---

## 3. Push & Wait

* Commit and push the workflow + README changes.
* GitHub Actions will run, generate the images, and commit them into your repo under `/images/`.
* Your profile README will then start showing the game 🎉

--------------------------------------------------------------------------------------------------------

## Run Instantly

When you commit the workflow for the first time, you will see the output after 24 hours due to the set cron job.
But you don’t need to wait 😃 — you can trigger it manually:

1. Go to your same profile repo (`github.com/<your-username>/<your-username>`).
2. Click **Actions** tab.
3. Select your workflow (likely called *“generate breakout svg”*).
4. On the right, there’s a **“Run workflow”** button (because we added `workflow_dispatch:` in your yml).
5. Click it → wait for a few seconds and it will run the job and generate the game after a minute or so.
6. Go to your profile page and you will see the game.

After the workflow finishes:

* Refresh your repo → you should see the new `images/` folder with the SVGs.
* Your profile README will then render the breakout game automatically.

⚡ Tip: if you don’t see the **Run workflow** button, make sure Actions are enabled for your repo (`Settings → Actions → General → Allow all actions`).


## Change Frequency

If you copy paste the exact workflow given above, it will run every 24 hrs due to the set cron job:

```yaml
on:
  schedule:
    - cron: "0 */24 * * *"
```

If you want the game to be updated on your every commit then replace the above cron job with this:

```yaml
name: generate breakout svg

on:
  push:
      branches:
        - main   # or "master" depending on your default branch
```
### Note

No need to worry about ever increasing storage because the way that workflow is written:

* It **always overwrites** the files `images/breakout-light.svg` and `images/breakout-dark.svg`.
* So your repo will only ever have **two files**, one for light and one for dark mode.
* Older versions aren’t kept — they’re replaced on every run.



