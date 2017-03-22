---
layout: article
title: I18N in React with Typescript * React-Intl
tags: [intellij react-native windows]
categories: articles
comments: true
excerpt: Internationalization with TypeScript & React-Intl
image:
  teaser: intellij/react-native-400x250.png
ads: true
---
 
Create a new project with the [TypeScript fork of create-react-app](https://github.com/wmonk/create-react-app-typescript).

```bash
$ create-react-app react-intl-ts-demo --scripts-version react-scripts-ts
```

We'll be using TypeScript, but unfortunately react-intl is a babel plugin.  So we'll need to 
be able to run babel:

```bash
$ npm install --save-dev  babel-cli babel-plugin-react-intl babel-preset-es2015 babel-preset-react
```

Then we'll install react-intl

```bash
$ npm install --save react-intl
```

And then we'll install something to manage the translations:

```bash
$ npm install --save-dev react-intl-translations-manager  
```

So at this point the `package.json` file should look like this:

```json
{
  "name": "react-intl-ts-demo",
  "version": "0.1.0",
  "private": true,
  "devDependencies": {
    "babel-cli": "^6.24.0",
    "babel-plugin-react-intl": "^2.3.1",
    "babel-preset-es2015": "^6.24.0",
    "babel-preset-react": "^6.23.0",
    "react-intl-translations-manager": "^4.0.1",
    "react-scripts-ts": "1.1.7"
  },
  "dependencies": {
    "@types/jest": "^19.2.2",
    "@types/node": "^7.0.8",
    "@types/react": "^15.0.17",
    "@types/react-dom": "^0.14.23",
    "react": "^15.4.2",
    "react-dom": "^15.4.2",
    "react-intl": "^2.2.3"
  },
  "scripts": {
    "start": "react-scripts-ts start",
    "build": "react-scripts-ts build",
    "test": "react-scripts-ts test --env=jsdom",
    "eject": "react-scripts-ts eject"
  }
}
```

Open up the `index.tsx` file and add in the `IntlProvider`.  The `IntlProvider` makes 
the `react-intl` library to components via the [React Context](https://facebook.github.io/react/docs/context.html):

Here we're hardcoding the locale to "en", but we'll return to this later.

```typescript
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import App from './App';
import './index.css';

import { IntlProvider } from 'react-intl';
const locale='en';

ReactDOM.render(
    <IntlProvider locale={locale}>
        <App />
    </IntlProvider>,
    document.getElementById('root') as HTMLElement
);
```

Now we can internationalize our `App.tsx` file.  Import `FormattedMessage` and wrap
the text:

```jsx
import * as React from 'react';
import './App.css';

import { FormattedMessage } from 'react-intl';

const logo = require('./logo.svg');

class App extends React.Component<null, null> {
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2><FormattedMessage id="app.welcome"
                                defaultMessage="Welcome to React"
                                description="Welcome Message" /></h2>
        </div>
        <p className="App-intro">
            <FormattedMessage id="app.to_get_started"
                              defaultMessage="To get started, edit {filename} and save to reload."
                              values={{ filename: "<code>src/App.tsx</code>" }}
                              description="Getting Started" />

        </p>
      </div>
    );
  }
}

export default App;
```

We're going to add three scripts to `package.json`:

    "trans:compile": "tsc -p .  --target ES6 --module es6 --jsx preserve --outDir extracted",
    "trans:extract": "babel  \"extracted/**/*.jsx\"",
    "trans:manage": "node scripts/translationRunner.js"

(The idea from this comes from [this github comment](https://github.com/yahoo/babel-plugin-react-intl/issues/48#issuecomment-254477940)).

1) Compile the `tsx` to ES2015 in a temporary directory called `extracted`.  (Make sure you also add
this to "exclude" in your `package.json`)
  
```
npm run trans:compile
```
 
2) To be able to run babel on react, we need a `.babelrc` file like this:

```json
{
  "presets": ["es2015", "react"],
  "plugins": [
    ["react-intl", {
      "messagesDir": "./src/translations"
    }]
  ]
}
```
 
Then extract all the default strings into our `src/translations/extracted` directory.
 
```bash
npm run trans:extract
```

You should now see a file `App.json` that looks like the following:

```json
[
  {
    "id": "app.welcome",
    "description": "Welcome Message",
    "defaultMessage": "Welcome to React"
  },
  {
    "id": "app.to_get_started",
    "description": "Getting Started",
    "defaultMessage": "To get started, edit {filename} and save to reload."
  }
]
```

3) The last step is to use [react-intl-translations-manager](https://github.com/GertjanReynaert/react-intl-translations-manager)
to generate the stub files.  Create a file `scripts/translationRunner.js` and set it up like this:

```javascript
// translationRunner.js
const manageTranslations = require('react-intl-translations-manager').default;

manageTranslations({
    messagesDirectory: 'src/translations/extracted/',
    translationsDirectory: 'src/translations/locales/',
    languages: ['fr', 'zh'], // any translation---don't include the default language
});
```

Then you can run `node scripts/translationRunner.js` directly, or via our npm script:

```
> npm run trans:manage

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
```

This creates two .json files per language---one for the translations, and one for whitelisting
messages that are causing [invalid warnings](https://github.com/GertjanReynaert/react-intl-translations-manager#usage).

Modify the French file to look like this (disclaimer: these come from Google Translate):

```json
// src/translations/locale/fr.json
{
  "app.to_get_started": "Pour commencer, modifier {filename} et enregistrer pour recharger.",
  "app.welcome": "Bienvenue sur React"
}
```

For this demo, we'll just hardcode a different locale.  But to change the locale in real code, you'll
need to set the `key` on IntlProvider.  (See [this](https://github.com/yahoo/react-intl/issues/243) for
more information.)

The French locale data will come from the [react-intl package](https://github.com/yahoo/react-intl/wiki/API#addlocaledata).

   
To hardcode the French locale, modify index.tsx to look like this:

```typescript
   import * as React from 'react';
   import * as ReactDOM from 'react-dom';
   import App from './App';
   import './index.css';
   
   import { IntlProvider, addLocaleData } from 'react-intl';
   
   const locale = 'fr';
   // load our messages
   const messages = require('./translations/locales/fr.json');
   // get the French locale information from the browser
   import fr = require('react-intl/locale-data/fr');
   addLocaleData(fr);
      
   ReactDOM.render(
       <IntlProvider locale={locale} messages={messages} key={locale}>
           <App />
       </IntlProvider>,
       document.getElementById('root') as HTMLElement
   );
```

Before we check our page, let's add in a date component to `index.tsx`:

```
   <FormattedDate value={new Date()}
       year='numeric'
       month='long'
       day='2-digit'/>
```

<figure>
 	<img src="/images/react-intl/react-i18n.png">
 	<figcaption>create-react-app-ts in French.</figcaption>
</figure>


## And...

Although you can use one of react-intl's "Formatted*" controls, sometimes you need to access
the API directly, such as from within an attribute.  For this you should inject
the `intl` object into the props of your control:

```typescript
import { intlShape, injectIntl, FormattedMessage } from "react-intl";
class _SomeControl extends React.Component<{}, any> {

    static propTypes: React.ValidationMap<any> = {
        intl: intlShape.isRequired
    };

// TODO finish this example

````





   
   
   
