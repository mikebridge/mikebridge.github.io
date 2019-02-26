+++
title = "I18N in React with Typescript & React-Intl"
categories = ["intellij", "react", "typescript", "i18n"]
description = "Internationalization with TypeScript & React-Intl"
date = "2017-03-21"
aliases = ["articles/typescript-i18n-react-intl"]
+++

Earlier this week I went in search of a React-friendly i18n library and I spent some time
experimenting with **[react-intl](https://github.com/yahoo/react-intl)**.  React-intl is based on 
**[FormatJS](https://formatjs.io/)**, which is a library for localizing numbers, dates, 
and strings.  My impression is that *react-intl* is a fairly small, practical library that is written to make i18n 
unobtrusive for developers.

> *Edit, May 1, 2017*
>
> Today I'm wondering whether it may have been a mistake to adopt *react-intl* over the alternatives.  At the
> moment it appears to be [in limbo](https://github.com/yahoo/react-intl/pull/918) after Yahoo's collapse.  

But although React-intl has [type definitions](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/react-intl),
it does not play particularly well with TypeScript.  It assumes there is a
single default language which a developer will put directly into the source code, and 
those strings will later be auto-extracted from the JavaScript code and handed off to 
translators.  Unfortunately, it's a [babel plugin](https://github.com/yahoo/babel-plugin-react-intl) that does 
the extraction, so TypeScripters will either have to use a [separate library](https://github.com/bang88/typescript-react-intl) 
to extract strings, or else they'll need to create a parallel TypeScript-to-ES2015 compilation 
step to use the native libraries.  I went with the latter, inspired 
by [this github thread](https://github.com/yahoo/babel-plugin-react-intl/issues/48#issuecomment-254477940), 
because I wanted to use [react-intl-translations-manager](https://github.com/GertjanReynaert/react-intl-translations-manager),
and the typescript-only library didn't extract the language strings in a format that the translation manager 
could use.

_**TL;DR** The code for this example is at 
[https://github.com/mikebridge/react-intl-ts-demo/](https://github.com/mikebridge/react-intl-ts-demo/)_.

## Setup

Create a new project with the [TypeScript fork of create-react-app](https://github.com/wmonk/create-react-app-typescript):

```bash
$ create-react-app react-intl-ts-demo --scripts-version react-scripts-ts
```

Then add **babel** with the react plugins, **react-intl** and the **react-intl-translations-manager**:

```bash
$ npm install --save-dev  babel-cli babel-plugin-react-intl babel-preset-es2015 babel-preset-react
$ npm install --save react-intl
$ npm install --save-dev react-intl-translations-manager  
```

So at this point the [`package.json`](https://github.com/mikebridge/react-intl-ts-demo/blob/master/package.json) file should look something like this:

<script src="https://gist.github.com/mikebridge/722823800a09493158c0214c20ff4d16.js"></script>

Add a [`.babelrc`](https://github.com/mikebridge/react-intl-ts-demo/blob/master/.babelrc) file to compile `JSX` files:

<script src="https://gist.github.com/mikebridge/fa3de45095fb8488f55efeb62275dc24.js"></script>

## Usage

Open up the `index.tsx` file and add in the `IntlProvider`.  The `IntlProvider` makes 
the `react-intl` functionality available to components via the [React Context](https://facebook.github.io/react/docs/context.html):

Here we're hardcoding the locale to "en" for the time being:

<script src="https://gist.github.com/mikebridge/61aad4b029ad7403ec4139a24286c5ed.js"></script>

Now we can localize the strings in [`App.tsx`](https://github.com/mikebridge/react-intl-ts-demo/blob/master/src/App.tsx) 
file.  Import `FormattedMessage` and wrap the text:

<script src="https://gist.github.com/mikebridge/af70bf14cab15525e98b8eabbc19c2a0.js"></script>

## Translation

I added these three scripts to [`package.json`](https://github.com/mikebridge/react-intl-ts-demo/blob/master/package.json).  In
production I'll probably write a bash script to clean up the artifacts, but in the meantime:

<pre>
    "trans:compile": "tsc -p .  --target ES6 --module es6 --jsx preserve --outDir extracted",
    "trans:extract": "babel  \"extracted/**/*.jsx\"",
    "trans:manage": "node scripts/translationRunner.js"
</pre>

**1)**  The first script compiles the `tsx` files to ES2015 in a temporary directory called `extracted`.  (Make sure you also add
this to "exclude" in your [`package.json`](https://github.com/mikebridge/react-intl-ts-demo/blob/master/package.json)
and to `.gitignore`)
  
<pre>
    $ npm run trans:compile
</pre>
 
**2)** The next one extracts all the default strings into the `src/translations/extracted` directory.
 
<pre>
    $ npm run trans:extract
</pre>

You should now see a file `App.json` that contains your extracted resource metadata:

<script src="https://gist.github.com/mikebridge/b0ab0954f71b8148e5cfc1ddf52d95b6.js"></script>

**3)** Lastly, [react-intl-translations-manager](https://github.com/GertjanReynaert/react-intl-translations-manager)
generates the stub files.  I copied their [`scripts/translationRunner.js`](https://github.com/mikebridge/react-intl-ts-demo/blob/master/scripts/translationRunner.js) script:

<script src="https://gist.github.com/mikebridge/6a07527d3e15b7b44755645866238a6a.js"></script>

Then you can run `node scripts/translationRunner.js` directly, or via the npm script:

<pre>
$ npm run trans:manage

Duplicate ids:

  No duplicate ids found, great!

Maintaining fr.json:

Added keys:
  app.to_get_started: To get started, edit {filename} and save to reload.
  app.welcome: Welcome to React

Maintaining zh.json:

Added keys:
  app.to_get_started: To get started, edit {filename} and save to reload.
  app.welcome: Welcome to React
</pre>

This creates two `.json` files per language---one for the translations, and one for whitelisting
messages that are causing [invalid warnings](https://github.com/GertjanReynaert/react-intl-translations-manager#usage).

Modify the French file to look like this (disclaimer: these come from Google Translate):

<script src="https://gist.github.com/mikebridge/cdbe03985dddc2ad421b1c7399bde8dd.js"></script>

For this demo, I just hardcoded a different locale to see it working.  But to change the locale in real code, you'll
need to set the `key` on IntlProvider.  (See [this](https://github.com/yahoo/react-intl/issues/243) for
more information---redux would be a good solution.)

The French locale data will come from the [react-intl package](https://github.com/yahoo/react-intl/wiki/API#addlocaledata).

To hardcode the French locale, modify index.tsx to look like this:

<script src="https://gist.github.com/mikebridge/b5fc53ec62e52f2cb9c67c2ec251ae0f.js"></script>

Before we check our page, let's add in a date component to `index.tsx`:

<pre>
   <FormattedDate value={new Date()}
       year='numeric'
       month='long'
       day='2-digit'/>
</pre>

Here's how it looks:

{{<figure src="/images/react-intl/react-i18n.png" caption="create-react-app-ts in French." caption-effect="fade" caption-position="bottom">}}

## Injecting react-intl into Components

Although you can use one of react-intl's "Formatted*" controls much of the time, some
text will occur inside TypeScript code or in JSX attributes.  Then you'll need to access the API 
directly. For this you can inject the `intl` object into the props of your control:

<script src="https://gist.github.com/mikebridge/728b3263aa878439b1d84876d23bdada.js"></script>

Those strings that don't appear in `Formatted*` controls still need to exist somewhere so that they can be
located during resource extraction.  There's a call `defineMessages` for this purpose—I wasn't sure what
to do with them so I put all these in a separate `.tsx` file:

<script src="https://gist.github.com/mikebridge/6a83c7fa6a0a3682e8897fb7116a432c.js"></script>

(I'm not sure what the object key here is for—I don't see it being used anywhere.)

## Conclusion

I am going to use `react-intl` for a smaller project and see how it goes.   I'll add more info
here as I get more practical experience with it.
   
   
   
