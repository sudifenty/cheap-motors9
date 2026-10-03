# Cheap Motors — Dealership Website

A car dealership website in ONE file: plain HTML, Tailwind CSS (CDN) and
vanilla JavaScript. No backend on this repository — the live inventory is a
small `cars.json` file that the owner publishes from the dashboard.

| File | What it is |
|---|---|
| `index.html` | The whole website AND the owner dashboard in one page. Customers see the car grid, photo galleries, search, contact buttons and showroom location. The **Owner Login** button (top right) opens the dashboard on the same page behind a passcode |
| `cars.json` | Created automatically by "Publish to Website" — the shared inventory your visitors see. Do not edit it by hand |
| `README.md` | This guide |

## Run it on your computer
Double-click `index.html`. Any modern browser works. Cars and photos are
saved in that browser's localStorage, and uploaded photos are compressed
automatically (max 900px, JPEG, up to 6 per car).

## Publish it with GitHub Desktop (step by step)

1. **Extract this ZIP first.** You end up with **one folder** that directly
   contains `index.html` and `README.md`. Do not drag the ZIP itself into
   GitHub Desktop — it can't read ZIPs.
2. Open **GitHub Desktop → File → Add Local Repository…** and select that folder.
   **Do not use "Clone"** — this repo is on your computer.
3. Click **Publish repository**. Name it e.g. `cheap-motors`, and **UNcheck
   "Keep this code private"** (free GitHub Pages needs a public repository).
4. Turn on the website: github.com → your repository → **Settings → Pages** →
   *Build and deployment* → **Source: Deploy from a branch** → **Branch: main**,
   **Folder: /(root)** → **Save**.
5. Wait 2–10 minutes, then visit `https://YOUR-USERNAME.github.io/REPO-NAME/`

## Adding cars (the owner's workflow)
1. Open your website and click **Owner Login** (password: set in `index.html`
   — see "Changing the dashboard password" below).
2. Add or edit cars — up to 6 photos each. Changes are saved in your browser
   immediately.
3. Click **Publish to Website**. This saves `cars.json` into this repository,
   and every visitor sees the same inventory after ~5–10 minutes.
   The first time it asks for your GitHub repository name and a fine-grained
   token (github.com → Settings → Developer settings → Fine-grained tokens →
   only this repository → Contents: Read and write). It is stored only in
   your browser.

## Changing the dashboard password
Open `index.html` in any text editor and find the line
`var CM_PASSCODE = 'admin123#';` near the bottom. Change the text between the
quotes, save, commit + push.

## About the password's limits (important)
This one-file site is hosted on GitHub Pages, which cannot run server code.
That means the dashboard password lives in the page — a technical visitor
could find it in the page source. It keeps casual visitors out, nothing more.
For a password that never touches the browser, use the separate PHP package
(`admin.php` + `logout.php` on PHP hosting): after you set it up and publish
once with its "Dashboard address" filled in, the website's Owner Login
button automatically sends you there instead.

## Seeing a 404? Check these in order
1. Wait a few minutes; check the repo's **Actions** tab.
2. `index.html` must be at the ROOT of the repo, never inside a folder.
3. GitHub Pages must be on (Settings → Pages → Branch: main, Folder: /root).
4. The repo must be public on a free plan.
5. Check the URL matches your username and repo name exactly.
