# hugo-mod-easy-head

[![standard-readme compliant](https://img.shields.io/badge/standard--readme-OK-green.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

A small [module](https://gohugo.io/hugo-modules/) for [Hugo](https://gohugo.io/) to simplify adding standard head elements to web pages.

Features provided by this module include:

    - generation of a `site.webmanifest` file for your site
    - generation of a `browserconfig.xml` file for your site
    - a partial for linking to a `.manifest` file
    - a partial for linking to a `browserconfig.xml` file
    - a partial for linking to a small set of standard icons
    - a partial for including verification tokens for Bing and Google

Note that this module does *not* generate or place icons for you. It just creates links.

## Table of Contents

- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [API](#api)
- [Maintainers](#maintainers)
- [Contributing](#contributing)
- [License](#license)

## Background

Every website that you build with Hugo will probably need to include some standard metadata files. It will also probably need to have some common head elements on each page. Setting this up for every site or theme you build gets tiresome, and may involve time-consuming and error-prone copy-pasting. This module aims to simplify the process.

Note that this is designed for simplicity, not flexibility. The 'easy' in the name is there for a reason. If you are developing PWAs or Windows web applications, the `site.webmanifest` and `browserconfig.xml` files created will probably be too simple for you.

This is also an *opinionated* project. Among its opinions are:

- manifest files should be called `site.webmanifest` and should live in your root directory
- almost all your icons should live in `/icons` (only `favicon.ico` lives in `/`)
- Andrey Sitnik's ideas about [which icons you need](https://evilmartians.com/chronicles/how-to-favicon-in-2021-six-files-that-fit-most-needs) are largely correct.

The module was originally implemented as multiple separate modules with names such as `easy_iconlinks` and `easy_verification`. As the number of modules created threatened to grow out of control, I've rolled them all into a single module. You're not obliged to use everything this module offers. You can pick and choose the parts that work for you. 

### Acknowledgements

The implementation is based largely on `@shedd`'s approach, described in [this Hugo forum thread](https://discourse.gohugo.io/t/solved-is-it-possible-to-use-site-variables-in-manifest-json/14018/6). It follows Andrey Sitnik's recommendations about [current best practices for favicons](https://evilmartians.com/chronicles/how-to-favicon-in-2021-six-files-that-fit-most-needs).

## Install

To use the module, add an import to your `config.toml` as follows:

```
[module]
  [[module.imports]]
    path = 'github.com/angusmci/hugo_mod_easy_head'
```

If you need to update the module, do:

```
hugo mod get -u
```

## Usage

### `site.manifest` and `browserconfig.xml` files

The `site.manifest` file is a simple description of some key elements of your file. The file itself is mainly intended to support progressive web apps (PWAs) but even if you aren't building a PWA, you'll probably want to provide a `site.manifest` file. This generates a very simple manifest suitable for a non-PWA site.

The `browserconfig.xml` file (sometimes called `ieconfig.xml`) was developed by Microsoft to support features of Internet Explorer. There is some reason to think that this file is obsolete, and that you don't need to include it any more. If you want a `browserconfig.xml`, however, this module will make one for you. Note that the generated file will contain links to some icons, which will live in your `/icons` directory. The icons aren't created for you; if you add a `browserconfig.xml` file, you'll need to create the icons yourself. 

To generate the `site.manifest` and `browserconfig.xml` files, add the following to your `config.toml` file.

```
[outputFormats]
    [outputFormats.manifest]
        name = "manifest"
        baseName = "site"
        mediaType = "application/manifest+json"
        notAlternative = "true"
    [outputFormats.browserconfig]
        name = "browserconfig"
        baseName = "browserconfig"
        mediaType = "application/xml"
        notAlternative = "true"

[outputs]
    home = [ "HTML", "RSS", "MANIFEST", "BROWSERCONFIG"]
```

If you want to generate one of the files but not the other, just remove the appropriate keyword from `[outputs]`.

The files are generated with sensible defaults, but you can fine-tune the output by adding some parameters to your `.Site.Params`. Parameters you might want to change are:

| Parameter         | Default  | Effect                                                                                  |
|-------------------|----------|-----------------------------------------------------------------------------------------|
| theme_color       | #FFFFFF  | Sets the `TileColor` in `browserconfig.xml` and the `theme_color` in `site.webmanifest` |
| background_color  | #FFFFFF  | Sets the `background_color` in `site.webmanifest`                                       |
| description       |          | If present, adds a `description` in `site.webmanifest`                                  |
| short_name        |          | If present, adds a `short_name` (max 12 characters) in `site.webmanifest`               |

For example, your `config.toml` file could contain:

```
title = "Amalgamated Bread Company"

[params]
short_name = "ABC"
description = "For all your bakery needs"
theme_color = "#FF0000"
background_color = "#FFEEEE"
```

### `easy-manifestlink`

To add a link to your manifest, use the `easy-manifestlink` partial in any layout that generates the `<head>` of your document, e.g.

```
{{ partial "easy-manifestlink" . }}
```

This will add the following tags to your `<head>`:

```
<link rel="manifest" href="/site.webmanifest" />
```

Note that this partial will only insert the manifest link on your site's homepage. You only need to link to the manifest once per site. Checkers like [`webhint`](https://webhint.io/) will complain if you include manifest links on other pages, so this partial takes care of making sure that the link shows up in the right place and nowhere else.

### `easy-browserconfiglink`

To add a link to your `browserconfig.xml`, use the `easy-browserconfiglink` partial in any layout that generates the `<head>` of your document, e.g.

```
{{ partial "easy-browserconfiglink" . }}
```

This will add the following tags to your `<head>`:

```
<meta name="msapplication-config" content="/browserconfig.xml" />
```

### `easy-iconlinks`

To link to the standard default icons, use the `easy-iconlinks` partial in any layout that generates the `<head>` of your document, e.g.

```
{{ partial "easy-iconlinks" . }}
```

This will add the following tags to your `<head>`:

```
<link rel="apple-touch-icon" href="/icons/apple-touch-icon.png" />
<link rel="icon" href="/favicon.ico" sizes="any" />
<link rel="icon" href="/icons/icon.svg" type="image/svg+xml" />
```

### Required icons

Between the various files and the `<head>` of your HTML documents, you will end up with links to the following icons:

| Location                    | Type | Size    | Linked from         |
|-----------------------------|------|---------|---------------------|
| /favicon.ico                | ICO  | 32x32   | `<head>`            |
| /icons/icon.svg             | SVG  | -       | `<head>`            |
| /icons/apple-touch-icon.png | PNG  | 180x180 | `<head>`            |
| /icons/icon-192.png         | PNG  | 192x192 | `site.webmanifest`  |
| /icons/icon-512.png         | PNG  | 512x512 | `site.webmanifest`  |
| /icons/smalltile.png        | PNG  | 70x70   | `browserconfig.xml` |
| /icons/mediumtile.png       | PNG  | 150x150 | `browserconfig.xml` |
| /icons/largetile.png        | PNG  | 310x310 | `browserconfig.xml` |
| /icons/widetile.png         | PNG  | 310x150 | `browserconfig.xml` |

To avoid cluttering your weblogs with 404s, you should ensure that appropriate icons exist at each of these locations.

### `easy-verification`

Tools such as [Google Search Console](https://search.google.com/search-console/) or [Microsoft Bing Webmaster Tools](https://www.bing.com/webmasters/) require that website owners prove their ownership of a website before the site can be managed by the tool. One method for doing this is to insert a special verification or validation token (unique to each site or owner) in the `<head>` of the site's homepage.

This module provides a simple way to add the verification codes to a site built using Hugo. It provides a single partial that can be invoked from whatever layouts generate `<head>` code for your site.

To use this, specify the validation codes that you want to use by adding them to your `config.toml` file or configuration directory as part of the site parameters.

```
[params]
google_verification = "..."
ms_verification = "..."
```

(Replace the '...' by the codes that you get from Google or Microsoft).

If you need different validation codes for production and development sites, look at the [Hugo documentation on configuration directories](https://gohugo.io/getting-started/configuration/#configuration-directory).

To add the relevant links to the `<head>` of your HTML documents, use the `easy-verification` partial in whatever layouts generate the `<head>` of your document, e.g.

```
{{ partial "easy-verification" . }}
```

### `easy-googleanalytics`

The `easy-googleanalytics` partial will inject Google Analytics tracking code into the head of your document. To use this, ensure that there is an entry for `google_analytics` in your site parameters, e.g.

```
[Params]
google_analytics = 'G-AA1234567'
```

Then include:

```
{{ partial "easy-googleanalytics" . }}
```

in your template, immediately after the opening `<head>` tag.

If no `google_analytics` value is defined in the site's parameters, no script code will be inserted.

### `easy-headlinks`

To simplify things still further, you can add most of the tags by invoking a single partial:

```
{{ partial "easy-headlinks" . }}
```

This will bring in all the partials defined in this module. You will still need to set up the `Site.Params` and `outputs` appropriately, however.

## Maintainers

[@angusmci](https://github.com/angusmci)

## Contributing

PRs accepted (but remember that the goal is to keep everything as simple as possible; if you want to add a million configuration parameters, fork the repo and call it `flexible-head` or something like that).

## License

MIT © 2022 Angus McIntyre






