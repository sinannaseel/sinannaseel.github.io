# Naseel Sinan - Personal Research Website

A modern, accessible personal research website built with Jekyll and hosted on GitHub Pages. This site showcases research in Task and Motion Planning (TAMP) for robotic manipulation in confined spaces, along with comprehensive guides and study notes.

## 🔗 Live Site

[sinannaseel.github.io](https://sinannaseel.github.io)

## 📋 Features

- **Modern Design**: Dark mode theme with responsive sidebar layout
- **Research Showcase**: Publications, posters, and research highlights
- **Comprehensive Guides**: 30+ tutorials covering robotics, Docker, Git, ML, and more
- **Study Notes**: Learning resources on advanced topics
- **Accessibility**: WCAG 2.1 compliant with keyboard navigation support
- **SEO Optimized**: Sitemap, robots.txt, and structured metadata
- **Automated Deployment**: GitHub Actions CI/CD pipeline

## 🛠️ Tech Stack

- **Static Site Generator**: [Jekyll](https://jekyllrb.com/)
- **Theme**: Minima with custom CSS
- **Hosting**: GitHub Pages
- **CI/CD**: GitHub Actions
- **Language**: Markdown, HTML5, CSS3

## 📦 Setup & Installation

### Prerequisites

- Ruby 3.0 or later
- Bundler
- Git

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/sinannaseel/sinannaseel.github.io.git
   cd sinannaseel.github.io
   ```

2. **Install dependencies**
   ```bash
   bundle install
   ```

3. **Build and serve locally**
   ```bash
   bundle exec jekyll serve
   ```

4. **View the site**
   Open your browser and navigate to `http://localhost:4000`

### Environment Variables

No environment variables required for basic development. For GitHub Pages deployment, the workflow uses the `GITHUB_TOKEN` automatically provided by GitHub Actions.

## 📂 Project Structure

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
├── _notes/                # Old notes folder (deprecated)
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
├── robots.txt             # SEO configuration
├── sitemap.xml            # XML sitemap
├── 404.md                 # Error page
└── Gemfile                # Ruby dependencies
```

## 📝 Content Management

### Adding a New Guide

1. Create a new markdown file in the `guides/` folder:
   ```markdown
   ---
   layout: default
   title: Your Guide Title
   ---

   # Your Guide Title

   Your guide content here...
   ```

2. Add an entry in `guides.md` with the proper link and description

3. Commit and push to trigger automatic deployment

### Adding Research/Publications

Edit `publications.md` or `research.md` with your content.

### Adding Study Notes

Create a new markdown file in the `notes/` folder with proper frontmatter (see guides for example).

## 🚀 Deployment

Deployment is **automatic** via GitHub Actions:

- **Trigger**: Push to `main` or `master` branch
- **Build**: Jekyll builds the static site
- **Deploy**: Site is published to GitHub Pages
- **Status**: Check the "Actions" tab in GitHub for build status

### Manual Build

To build the site locally without serving:
```bash
bundle exec jekyll build
```

Output will be in the `_site/` folder.

## ♿ Accessibility

This site follows **WCAG 2.1 Level AA** guidelines:

- ✅ Keyboard navigation support (Tab, Enter)
- ✅ Skip-to-content link
- ✅ ARIA labels on interactive elements
- ✅ Color contrast ratios meet standards
- ✅ Responsive design for all screen sizes
- ✅ Prefers-reduced-motion support
- ✅ Semantic HTML structure

## 🔍 SEO

The site includes:

- `robots.txt` - Search engine directives
- `sitemap.xml` - Complete site structure
- `jekyll-seo-tag` plugin - Meta tags and structured data
- Open Graph tags for social sharing
- Descriptive titles and meta descriptions

## 🎨 Customization

### Colors & Styling

Edit `assets/css/main.css` to customize:

- CSS Variables (`:root` section) for colors
- Grid layouts and responsive design
- Component styles (sidebar, cards, navigation)

### Configuration

Edit `_config.yaml` to customize:

- Site title and description
- Author information
- Social links (GitHub, LinkedIn, etc.)
- Navigation menu items

## 🐛 Troubleshooting

### Build fails locally

```bash
# Clean and rebuild
bundle exec jekyll clean
bundle exec jekyll build
```

### Port 4000 in use

```bash
bundle exec jekyll serve --port 4001
```

### Dependencies outdated

```bash
bundle update
```

### Site not updating on GitHub Pages

Check:
1. GitHub Actions workflow status (Settings → Actions)
2. Build logs for errors
3. Correct branch name in workflow (main/master)

## 📄 License

This project is licensed under the **MIT License** - see [LICENSE](LICENSE) file for details.

## 👤 Author

**Naseel Sinan**
- 📧 Email: [naseelsinan10@gmail.com](mailto:naseelsinan10@gmail.com)
- 🔗 GitHub: [@sinannaseel](https://github.com/sinannaseel)
- 💼 LinkedIn: [naseel-sinan](https://linkedin.com/in/naseel-sinan)
- 🏢 Affiliation: University of Manchester

## 🙏 Acknowledgments

- Advisor: [Dr. Murilo Marinho](https://mmmarinho.github.io/)
- Co-supervisor: [Dr. Tingting Mu](https://personalpages.manchester.ac.uk/staff/tingting.mu/)
- Facility: [RAICo](https://raico.org/) - Robotics Facility at Whitehaven

## 📚 Resources

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [Minima Theme](https://github.com/jekyll/minima)
- [GitHub Pages Docs](https://docs.github.com/en/pages)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)

---

**Last Updated**: February 2026
