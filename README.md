# Portfolio MkDocs

Hello ! this project is a portfolio website built using MkDocs, designed to showcase projects and provide information about the portfolio owner. The site is structured to be easily navigable and visually appealing.

## Project Structure

```
portfolio-mkdocs
├── docs
│   ├── index.md        # Homepage of the portfolio
│   ├── about.md        # Information about the portfolio owner
│   └── projects.md     # Showcase of completed projects
├── .github
│   └── workflows
│       └── deploy.yml  # GitHub Actions workflow for deployment
├── mkdocs.yml          # MkDocs configuration file
└── README.md           # Project overview and setup instructions
```

## Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/portfolio-mkdocs.git
   cd portfolio-mkdocs
   ```

2. **Install MkDocs:**
   Ensure you have Python and pip installed, then run:
   ```bash
   pip install mkdocs
   ```

3. **Run the development server:**
   Start the MkDocs development server to preview your site:
   ```bash
   mkdocs serve
   ```
   Open your browser and go to `http://127.0.0.1:8000` to view your portfolio.

4. **Build the site:**
   To build the static site for deployment, run:
   ```bash
   mkdocs build
   ```

5. **Deploy to GitHub Pages:**
   The project includes a GitHub Actions workflow that automatically deploys the site to GitHub Pages when changes are pushed to the main branch.

## Contributing

Feel free to fork the repository and submit pull requests for any improvements or features you would like to add. 

## License

This project is licensed under the MIT License. See the LICENSE file for details.