LocalStack Blog
==================

The blog uses [Hugo](https://github.com/gohugoio/hugo) and the [doks theme](https://github.com/h-enk/doks).

## Build and run locally

Make sure you have `npm` and `hugo` (extended - for sass compilation) installed. A VSCode devcontainer setup is available with all required tools already preconfigured.

### Install dependencies

npm is used to manage the frontend dependencies.

    npm install

### Using the make file

A Make file is available with a `dev` target that runs hugo in watch mode and serves the website for preview purposes.
Start it by executing

```
make dev
```

in your terminal

### Build the static website

    npx hugo --gc --minify

And find the results in `public/`

### Run development server

... and to start the server

    npm run server

Or alternatively (`-D` activates blog drafts)

    hugo server --watch=true --disableFastRender -D

and navigate to http://localhost:1313

## Deploy

Pushing to `main` will trigger a github actions workflow that deploys the website via the `gh-pages` branch to github pages.

# Migration Notice

- The website has been migrated to another service and the blog was forked from the old repo https://github.com/localstack/localstack.github.io
- The blog is now served at `/` on `blog.localstack.cloud`.
- Auxiliary files that were previously hosted on the website are now hosted in an external CDN.
