# Documentation for the 42 Minishell project.

This repository contains documentation based on our research for Minishell (initially, to give my teammate and me a common thread to follow in case we got a bit lost ^^).

This documentation was created using a software called Obsidian. This tool, among other things, is useful for:

- Linking all our technical notes together (like a mini-Wikipedia) through a very simple linking system. If we talk about the Executor, you can click to land directly on the Executor-related note.

- Visualizing our architecture (Well, not sure the graph is actually useful for that xD).

- Managing our progress with a Kanban board to track our tasks (To Do, In Progress, Done) directly within our documentation.

- Staying compatible with GitHub. Since it's pure Markdown, the entirety of this documentation remains perfectly readable on GitHub!


---

### Obsidian Installation

If you want to open this vault (the folder where Obsidian stores our notes), you just need to install Obsidian, available on [Obsidian.md](https://obsidian.md/).

Download the version for your OS and install it!

At launch, click on **Open folder as vault** and select the root folder of the Git repository.

---

### Enabling the "Kanban" Plugin

By default, Obsidian does not have a Kanban mode; it's a community plugin provided by **mgmeyers**.

To be able to use Kanban, you will need to:

- Open the **Settings** (the gear icon at the bottom left).

- In the left menu, go to **Community plugins**.

- Disable **Restricted Mode** -> Click the **Turn on community plugins** button.

- Then click on **Browse**.

- Search for **Kanban** in the search bar.

- Click on the **Kanban** plugin (by mgmeyers) and install it.

- Once installed, simply activate it (**Enable**).


If you want to use the plugin on your own to create other boards, press `Ctrl + P` to open the command palette and type `Kanban`. After that, just choose `Kanban: Create new board` and give it a name.

---

### Documentation Structure

```
Minishell-Vault/
├── .obsidian/          <-- Obsidian's folder (hidden on Mac/Linux)
├── 00_Management/      <-- Dashboard, Routine, Bug log, and Tags.
├── 10_Research/        <-- Theoretical documentation (Lexer, Parser...)
├── 20_Specs/           <-- Technical specifications (C Structures, Prototypes...)
├── 30_Resources/       <-- Useful PDFs and links used
├── README.md           <-- The same you are currently reading, but in French !
├── README_EN.md        <-- What you are currently reading
└── Roadmap             <-- The Roadmap
```

---
