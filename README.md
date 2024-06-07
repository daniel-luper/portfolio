# My Personal Blog & Portfolio

A lightweight blog and portfolio website, built with [SvelteKit](https://kit.svelte.dev). 
It's deployed on [Vercel](https://vercel.com), and you can see it live [here](https://danluper.com).

It was built with a few goals in mind:

- Intuitive: easy to navigate and understand
- Fast: only load what's needed
- Pretty: a pleasant design that is both accessible and pleasing to the eye
- Responsive design: look and behave well on all screen sizes
- Simple but flexible blog-writing: write posts in Markdown with support for Svelte components

## 📝 Managing Posts

All posts are Markdown files that are processed with [MDsveX](https://mdsvex.pngwn.io/) to allow using Svelte components inside them. In order to make it easier to manage posts, you can use the [Front Matter VS Code extension](https://frontmatter.codes/), which gives you a nice CMS-like UI.

## 🛠️ Developer's Guide

### Building & Running Locally

To run it locally, you simply have to run:

```shell
# First, install dependencies
npm install
# Then, run it on dev mode
npm run dev
```

The site should now be available at http://localhost:5173/ on your local machine, and your local 
machine's IP address on your network—great for testing on mobile OSes.

### Histoire / Storybook

This project uses [Histoire](https://histoire.dev), a Vite-based Storybook alternative to be able 
to see and develop components in isolation. To open it, run `npm run story:dev`.

## 🖼️ Image Optimization

This website uses [image-transmutation](https://github.com/matfantinel/image-transmutation) to automatically optimize images used in the site. This means that even if you use non-optimal image formats (like lossless PNGs), it will go over the images and convert images to WebP and AVIF for you, as long as you use the `<Image />` component instead of `<img />`. This is done on build, so it doesn't change anything when running the website locally.

## 📜 License and Credits
This project was adapted from [sveltekit-static-blog-template](https://https://github.com/matfantinel/sveltekit-static-blog-template) by [Matt Fantinel](https://fantinel.dev).

The code is open source and available under the [GPL-3.0 License](LICENSE).