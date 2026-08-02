# GitHub Pages Deployment

This project is ready to deploy with GitHub Pages through GitHub Actions.

## 1. Create a GitHub repository

Create a new repository under your own account, for example:

- `aaaloooooof.github.io` for a root site at `https://aaaloooooof.github.io/`
- `personal-website` for a project site at `https://aaaloooooof.github.io/personal-website/`

## 2. Upload this project

Push the whole project directory to your repository, including `.github/workflows/deploy-pages.yml`.

## 3. Turn on GitHub Pages

In your GitHub repository:

1. Open `Settings`
2. Open `Pages`
3. Set `Source` to `GitHub Actions`

## 4. Trigger deployment

Push to the `main` branch. GitHub Actions will automatically:

- install dependencies
- build the site into `dist/`
- publish `dist/` to GitHub Pages

## 5. Replace placeholder metadata

Before or after the first push, update these fields in `package.json`:

- `homepage`
- `repository`
- `bugs`

Replace:

- `YOUR_GITHUB_USERNAME`
- `YOUR_REPOSITORY_NAME`

with your real GitHub username and repository name.

## 6. Final website URL

Your final URL will usually be one of these:

- `https://YOUR_GITHUB_USERNAME.github.io/`
- `https://YOUR_GITHUB_USERNAME.github.io/YOUR_REPOSITORY_NAME/`

Once you have that final URL, generate a QR code from it and mobile devices will be able to open your homepage directly.
