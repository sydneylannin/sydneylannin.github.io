# Sydney Lannin · Portfolio

Live site: https://sydneylannin.github.io

## Adding a new project

1. Open the `projects` folder and click **`_template.md`**. Copy everything in it.
2. In the `projects` folder, click **Add file → Create new file**. Name it something like `my-new-project.md` (lowercase, dashes instead of spaces). Paste the template in.
3. Fill in the settings at the top: title, card text, and what goes on the left side (an embed link, a PDF, screenshots, buttons). Then write your post below the second `---` line.
4. Open **`projects/list.txt`**, add `my-new-project` on its own line where you want the card to appear, and commit.

The site updates within a couple of minutes.

### Adding pictures or PDFs

- **Screenshots:** open `images/projects`, click **Add file → Upload files**, and drag them in. Then add a line like this to the project's settings:
  `image: images/projects/my-screenshot.png | Caption text`
- **Card picture:** same as above, then add `card-image: images/projects/my-card.jpg`.
- **PDFs** (papers, slides): upload them to the `files` folder and add:
  `document: files/my-paper.pdf` (shows the pages on the project page)
  `link: Open the full paper | files/my-paper.pdf` (adds a button)

## Editing what's already here

- **A project's write-up:** open its `.md` file in `projects` and click the pencil icon. Anything inside `<mark>…</mark>` shows with a yellow highlight. Those are prompts to replace with your own writing.
- **Hide a project without deleting it:** put a `#` in front of its name in `projects/list.txt`.
- **Home / About / Contact text:** in `index.html`, near the top of the `<body>`. Remove `class="ph"` once you've replaced a highlighted paragraph.
- **Resume:** upload a new PDF named exactly `Lannin_Sydney_Resume.pdf` into the `files` folder. It replaces the old one.

## Writing in Markdown (the .md files)

| You type | You get |
|---|---|
| `## Heading` | a section heading |
| `- item` | a bullet point |
| `**bold**` / `*italic*` | **bold** / *italic* |
| `[text](https://link.com)` | a link |
| a blank line | a new paragraph |
