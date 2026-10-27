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
    - a partial for including the robots meta tag
    - a partial for adding Google Analytics code
    - a partial for linking to the `humans.txt` file.

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

### `easy-head`

The simplest way to use this module is to add a single partial to the `<head>` of your HTML documents:

```
{{ partial "easy-head" . }}
```

This will bring in the other partials defined in this module, adding a large set of standard tags to your `<head>`. The default settings will probably give you sensible results for a basic website, but you may still need to do some basic configuration through `Site.Params` and `outputs` to get the exact behavior you want. You're also responsible for ensuring that you have any required supporting files, e.g. icons or a "humans.txt" file.

The `easy-head` partial will also insert a call to `hugo.Generator` to add a generator tag on the homepage. If you would prefer not to have this for any reason, use Hugo's [`disableGeneratorInject`](https://gohugo.io/configuration/all/#disablehugogeneratorinject) site parameter to disable it.

Read the descriptions of the individual partials below to see how to configure them.

If you want to fine-tune the contents of your `<head>` a bit more, instead of using `easy-head` you can use any combination of the individual partials listed below separately. For example, if you don't like the set of recommended icon links inserted by `easy-iconlinks`, you could use the other partials to set up the `<head>` the way you want it, and then add your own set of iconlinks.

### `easy-manifestlink`

The `easy-manifestlink` partial is invoked with

```
{{ partial "easy-manifestlink" . }}
```

to add a link to your [web application manifest](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Manifest). It will add the following tags to your `<head>`:

```
<link rel="manifest" href="/site.webmanifest" />
```

Note that this partial will only insert the manifest link on your site's homepage, not on other pages. You only need to link to the manifest once per site. Checkers like [`webhint`](https://webhint.io/) will complain if you include manifest links on other pages, so this partial takes care of making sure that the link shows up in the right place and nowhere else.

### `easy-browserconfiglink`

To add a link to your `browserconfig.xml`, set the parameter `enableBrowserConfigLink` to `true` in your site parameters and include either `easy-head` or `easy-browserconfiglink` in the `head` of your document.

```
{{ partial "easy-browserconfiglink" . }}
```

This will add the following tags to your `<head>`:

```
<meta name="msapplication-config" content="/browserconfig.xml" />
```

### `easy-iconlinks`

To link to the standard default icons, include either `easy-head` or `easy-iconlinks` in any layout that generates the `<head>` of your document, e.g.

```
{{ partial "easy-iconlinks" . }}
```

This will add the following tags to your `<head>`:

```
<link rel="apple-touch-icon" href="/icons/apple-touch-icon.png" />
<link rel="icon" href="/favicon.ico" sizes="any" />
<link rel="icon" href="/icons/icon.svg" type="image/svg+xml" />
```

### `easy-verification`

Tools such as [Google Search Console](https://search.google.com/search-console/) or [Microsoft Bing Webmaster Tools](https://www.bing.com/webmasters/) require that website owners prove their ownership of a website before the site can be managed by the tool. One method for doing this is to insert a special verification or validation token (unique to each site or owner) in the `<head>` of the site's homepage.

This module provides a simple way to add the verification codes to a site built using Hugo. It provides a single partial that can be invoked from whatever layouts generate `<head>` code for your site.

To use this, specify the validation codes that you want to use by adding them to your `config.toml` file or configuration directory as part of the site parameters.

```
[params]
googleVerification = "..."
msVerification = "..."
```

(Replace the '...' by the codes that you get from Google or Microsoft).

If you need different validation codes for production and development sites, look at the [Hugo documentation on configuration directories](https://gohugo.io/getting-started/configuration/#configuration-directory).

To add the relevant links to the `<head>` of your HTML documents, use the `easy-head` or `easy-verification` partials in whatever layouts generate the `<head>` of your document, e.g.

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

or `easy-head` in your template. Google suggests placing this immediately after the opening `<head>` tag. If you use the `easy-head` partial, then the Google Analytics code may not be the very first thing after the `<head>` but it will be close to the top.

If no `google_analytics` value is defined in the site's parameters, no script code will be inserted.

### `easy-authormeta`

The `easy-authormeta` partial will inject a meta element to show the name of the author of the page, e.g.

```
<meta name="author" content="Jane Smith">
```

The name is taken either from the `author` param of the page or of the site (in the sitewide config file), e.g.

```
[Params]
author = Jane Doe
```

If there is no `author` param in either the page frontmatter or the site configuration, the meta tag will not be generated.

### `easy-robotsmeta`

The `easy-robotsmeta` partial will inject a element tag to control indexing by robots, e.g.

```
<meta name="robots" content="index, nofollow">
```

The value is taken either from the `robots` param of the page or of the site (in the sitewide config file), e.g.

```
[Params]
robots = noindex, nofollow
```

If you don't provide a `robots` param, the meta element will not be generated.

 See Google's page on the [robots meta tag](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag) for a list of possible values.

### `easy-humanslink`

If your site has a [humans.txt](https://humanstxt.org/) file to describe the creators, then set `enableHumansLink` to `true` in your site configuration.

```
[Params]
enableHumansLink = true
```

in the site configuration will add:

```
<link rel="author" href="/humans.txt" type="text/plain">
```

to the `<head>` of your document.

### `easy-viewport`

This partial adds a `viewport` `<meta>` element to the head. The default element generated will look like:

```
<meta name="viewport" content='width=device-width,initial-scale=1'>
```

but if you need to customize it, add a `viewport` parameter to the page or site configuration.

### `easy-keywords`

This partial adds a `keywords` `<meta>` element, such as:

```
<meta name="keywords" content='keyword1, keyword2' lang='en-us'>
```

to the `<head>` of the document. Keywords are defined in the frontmatter of the page and accessed via the `.Keyword` method.  The language is set based on the `.Language.Locale` method.

### `easy-description`

This partial adds a `description` `<meta>` element, such as:

```
<meta name="description" content='This is a description of this page.' lang='en-us'>
```

to the `<head>` of the document. The description is defined in the frontmatter of the page and accessed via the `.Description` method. The language is set based on the `.Language.Locale` method.

## Other

### Required icons

Between the various external files and the `<head>` of your HTML documents, you will end up with links to most or all of the following icons:

| Location                    | Type | Size    | Linked from         |
|-----------------------------|------|---------|---------------------|
| /favicon.ico                | ICO  | 32x32   | `<head>`            |
| /icons/icon.svg             | SVG  | -       | `<head>`            |
| /icons/apple-touch-icon.png | PNG  | 180x180 | `<head>`            |
| /icons/icon-192.png         | PNG  | 192x192 | `site.webmanifest`  |
| /icons/icon-512.png         | PNG  | 512x512 | `site.webmanifest`  |
| /icons/icon-mask.png        | PNG  | 512x512 | `site.webmanifest`  |
| /icons/smalltile.png        | PNG  | 70x70   | `browserconfig.xml` |
| /icons/mediumtile.png       | PNG  | 150x150 | `browserconfig.xml` |
| /icons/largetile.png        | PNG  | 310x310 | `browserconfig.xml` |
| /icons/widetile.png         | PNG  | 310x150 | `browserconfig.xml` |

To avoid cluttering your weblogs with 404s, you should ensure that appropriate icons exist at each of these locations. This module does not include icons or generate them for you.

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

## Maintainers

[@angusmci](https://github.com/angusmci)

## Contributing

PRs accepted (but remember that the goal is to keep everything as simple as possible; if you want to add a million configuration parameters, fork the repo and call it `flexible-head` or something like that).

## License

MIT © 2022 Angus McIntyre






