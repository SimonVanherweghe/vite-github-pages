# Deploying a Vite project to GitHub Pages

The plain HTML page went online by pushing it: the files you wrote were the site. A Vite project is
different. What you write is not what the browser gets: `npm run build` turns it into a `dist`
folder, and only that folder belongs online. So somebody has to run the build. Not you, on your
laptop, but a clean machine, every time you push. That is what a GitHub Actions workflow does.

This is the fake-door page from the Vite chapter. You put it on GitHub Pages, built by a workflow.

Useful resources:

- [Deploying a static site, GitHub Pages](https://vite.dev/guide/static-deploy#github-pages) in the Vite docs
- [Public base path](https://vite.dev/guide/build#public-base-path)
- [Understanding GitHub Actions](https://docs.github.com/en/actions/get-started/understand-github-actions)

The most important steps are described below.

## 1. Build it once yourself

Install the dependencies and start the dev server, to check that everything works:

```bash
npm install
npm run dev
```

Then build it:

```bash
npm run build
```

Open the new `dist` folder and compare it with your source. Your `css` folder, `main.js` and the
`js-confetti` package from `node_modules` are now one JavaScript file and one stylesheet in
`dist/assets`, with a hash in their names. That folder is what goes online. `dist` is in
`.gitignore`, so it is never committed: the workflow builds it again on GitHub.

## 2. Create a repository and push

Create a new **private** repository on GitHub called `fake-door`, like all your work in this
course, and push this project to it.

> [!NOTE]
> GitHub Pages from a private repo needs GitHub Pro, which you get for free with GitHub Education.
> Can't pick *GitHub Actions* in step 5? Your Education benefits are not active yet: tell us in
> class. The repo stays private; the site itself is public.

## 3. Set the base path

Your site will live at `https://<your-username>.github.io/fake-door/`: in a subfolder named after
your repository, not at the root of the domain. Vite writes asset paths that start at the root
(`/assets/index-....js`), so online the browser looks for them in the wrong place and your page
loads without CSS or JavaScript.

Tell Vite where the site will live. Create `vite.config.js` next to `package.json`:

```js
export default {
  base: '/fake-door/'
}
```

The value is your repository name between two slashes. Is your repo called something else? Use
that name. Build again and open `dist/index.html`: every asset path now starts with `/fake-door/`.

## 4. Add the workflow

Create the file `.github/workflows/deploy.yml` (mind the dot in front of `.github`) and paste in the
workflow from the
[GitHub Pages section of the Vite docs](https://vite.dev/guide/static-deploy#github-pages). Copy it
from the docs rather than from an old project: the version numbers of the actions change.

Read it before you commit it. Every step is something you would do yourself on a brand-new
computer:

| Step | What it does |
| --- | --- |
| `on: push: branches: ['main']` | run on every push to `main` |
| `runs-on: ubuntu-latest` | on a fresh Linux machine at GitHub |
| `actions/checkout` | get your code |
| `actions/setup-node` | install Node |
| `npm ci` | install exactly what `package-lock.json` says |
| `npm run build` | make `dist` |
| `actions/upload-pages-artifact` with `path: './dist'` | hand over `dist`, and nothing else |
| `actions/deploy-pages` | put it online |

`uses:` runs an action someone else wrote and published. `run:` types a command in that machine's
terminal.

Commit the config and the workflow, and push.

## 5. Switch Pages to GitHub Actions

On GitHub, go to your repository's **Settings → Pages**. Under **Build and deployment**, set
**Source** to **GitHub Actions**, not *Deploy from a branch*.

## 6. Follow the run

Open the **Actions** tab. Your push started a run: a yellow dot while it builds, a green tick when
it is done. Then visit `https://<your-username>.github.io/fake-door/`.

A red cross? Click the run, then the step that failed, and read the last lines of its log. The
usual suspects:

- **`npm ci` fails**: `package-lock.json` is not committed.
- **The page loads, but without CSS**: `base` is missing or does not match the repository name.
- **A file is not found**, while it works on your laptop: the case of a file name differs from the
  import. macOS does not care about `Door.png` against `door.png`; the Linux machine does.

## 7. Change something

Change the heading in `index.html`, commit, push. Watch the Actions tab: a new run starts on its
own, and a minute later the change is online. You never ran `npm run build` for it.

That is the whole point. Astro works exactly the same way, with one action that packs the Node,
install, build and upload steps together.
