## Local Development

**Install dependencies**
```bash
bundle install
```

**Build and serve locally**
```bash
bundle exec jekyll serve
```

Open your browser and navigate to `http://localhost:4000`

## Project Structure

```
.
├── _config.yaml           # Jekyll configuration
├── _layouts/              # HTML templates
│   ├── default.html       # Main page layout
│   ├── page.html          # Page template
│   └── note.html          # Note template
├── _includes/             # Reusable HTML components
│   ├── header.html        # Site header and navigation
│   ├── footer.html        # Site footer
│   └── navigation.html    # Navigation menu
├── assets/                # Static assets
│   ├── css/
│   │   └── main.css       # Styles
│   └── images/            # Images and graphics
├── guides/                # Technical guides and tutorials
├── notes/                 # Study notes collection
├── index.md               # Homepage
├── guides.md              # Guides index
├── notes.md               # Notes index
├── research.md            # Research overview
├── publications.md        # Publications list
├── contact.md             # Contact information
├── 404.md                 # Error page
└── Gemfile                # Ruby dependencies
```
