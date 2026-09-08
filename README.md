# Social Finder

A small local Flask tool that finds a website's official social media links, and can cross-check a username against a curated list of other platforms to see where else it's registered.

Runs in batch mode: paste multiple URLs/usernames (or upload a `.txt`/`.csv`), and it processes them one at a time, with results viewable in the browser and exportable as CSV or JSON.

## Features

- **Scrape a site for its social links** — checks the homepage plus common `/about` and `/contact` pages, prioritizing header/footer/nav areas and page metadata (`sameAs`, Twitter meta tags) over links buried in body content.
- **Cross-platform username probing** — given a bare username, checks it against a curated set of platforms (GitHub, GitLab, Twitch, Steam, etc.) known to reliably distinguish a real profile from a fake one over a plain HTTP request.
- **Batch mode** — paste multiple entries or upload a file; processed sequentially.
- **Export** — download results as CSV or JSON after a batch run.

## Setup

```bash
git clone <this-repo-url>
cd social-finder-multi
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Then open **http://127.0.0.1:5000** in your browser.

## Usage

1. Paste one URL or username per line into the textarea, and/or upload a `.txt`/`.csv` file of entries (one per row/line). Both can be combined; duplicates are removed automatically.
2. Click **Run batch**. Entries are processed one at a time — larger batches take longer.
3. Review results per entry: links found directly on the site, plus any "same username elsewhere" matches.
4. Use **Download CSV** / **Download JSON** to export the results of the most recent run.

Batches are capped at 25 entries per submission (see `MAX_BATCH_SIZE` in `app.py`) to keep a single run from taking too long or hammering too many external sites at once. Adjust as needed for your own use.

## How it decides a platform match is real

For scraping a site's own links, the crawler pattern-matches against known social profile URL shapes, then filters out generic path segments (`/login`, `/about`, `/pricing`, etc.) that would otherwise look like a valid "handle."

For the username cross-check, only platforms that were manually verified to return a genuinely different response for real vs. fake usernames are included — most social apps render the same generic 200 OK page for both, since the "not found" state only shows up after client-side JavaScript runs, which a plain HTTP request never triggers. See the comments above `PROFILE_URL_TEMPLATES` in `app.py` for the specific platforms that were tested and excluded, and why.

## Security notes

- The app binds to `127.0.0.1` (localhost only) by default. **Do not** change this to `0.0.0.0` while `debug=True` is set — Flask's debug mode exposes an interactive code-execution console on unhandled errors, and binding to all interfaces would make that reachable from your local network.
- This is a Flask **development server**, not meant for production or public deployment. If you ever want to run this somewhere other people can reach, put it behind a proper WSGI server (Gunicorn, uWSGI) and a reverse proxy, and turn debug mode off.
- Results (including exported CSV/JSON) are held in memory for the current run only — nothing is persisted to disk or a database.

## Responsible use

This tool only reads publicly accessible pages and makes the kind of requests a browser or curious human would make. That said:

- The username cross-check tells you *a username exists elsewhere*, not that it belongs to the same person — treat matches as leads, not confirmation.
- Please don't use this to look up people without their knowledge or consent, or to build profiles on people for harassment, stalking, or doxxing. Some intentionally-excluded platforms (see `app.py` comments) reflect that same principle.
- Respect the target sites' terms of service and robots.txt where applicable, and avoid hammering any one site with excessive requests.

## License

MIT — see [LICENSE](LICENSE).
