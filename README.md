# Organize

Organize is an iOS to-do list manager built in LiveCode. The repository is prepared for manual, script-only development so features can be added iteratively while keeping a clean Git history.

## Repository Structure

- `src/` – Script-only stack files (`.livecodescript`). Add new stacks here so their code is easy to review and merge.
- `assets/` – Fonts, icons, images, sounds, and other shared resources. Sub‑folders are pre-created with placeholders.
- `widgets/` – LiveCode extensions and widgets. The bundled calendar widget can be installed from this folder when needed.
- `stacks/` – Binary stack files that you may generate temporarily during development or prototyping.
- `build/` – Output from iOS standalone builds. Git ignores this directory except for the placeholder file.
- `docs/` – Reference materials, including screenshots, data grid behaviour scripts, and database planning notes.

## Development Approach

This project is intentionally configured for manual LiveCode development with Git version control:

- Prefer script-only stacks saved into `src/` so changes appear as readable text diffs.
- Keep binary stacks out of version control unless absolutely necessary; if you must create one, place it in `stacks/`.
- Store reusable extensions in `widgets/` and import them into the LiveCode IDE as required.
- Treat assets as shared resources that can be referenced by both the IDE and build pipeline.

## Getting Started

1. **Install LiveCode** – Download LiveCode 9.6 (or later) from [livecode.com](https://livecode.com/).
2. **Clone the repository** – `git clone <repo-url>` and check out the branch you plan to work on.
3. **Create or open stacks** – Launch LiveCode and use `File → New Stack` (for prototyping) or `File → New Script Only Stack` to add `.livecodescript` files under `src/`.
4. **Manage widgets** – Import the calendar widget by opening the Extension Manager and pointing it to `widgets/com.livecode.widget.calendar.1.0.0`.
5. **Build when ready** – Place any generated standalone builds inside `build/`; Git will ignore the compiled artifacts automatically.

Refer to the documents in `docs/` for UI references, code snippets, and data grid behaviour scripts as you extend the application.
