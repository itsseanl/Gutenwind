# Gutenwind 

Integrating TailwindCSS V4 with an example gutenberg block scaffolded with @wordpress/create-block tool.

There is probably a better way to do this, but this is how I got it working after a day of pain and suffering.

## Step 1 - create block

Scaffold the block with create-block 

`npx @wordpress/create-block@latest block-title`

## Step 2 - Install TailwindCSS

I achieved this via a modified version of the [install using postCSS](https://tailwindcss.com/docs/installation/using-postcss) method.


First, install dependencies: 
`npm install tailwindcss @tailwindcss/postcss postcss`

## Step 3 - Modify default webpack.config.js

Next, we're going to modify the existing webpack.config.js to accomodate the @tailwindcss/postcss module.

The default webpack.config.js that ships with create-block can be found in `node_modules/@wordpress/scripts/config/webpack.config.js`. Copy this into the root directory of your newly created plugin.

There are a few require statements with relative paths you'll have to adjust.

```
[line 25] const PhpFilePathsPlugin = require( '../plugins/php-file-paths-plugin' );
[line 26]const RtlCssPlugin = require( '../plugins/rtlcss-webpack-plugin' );
...
[line 39] ...} = require( '../utils' );
```

Change these to 

```
[line 25] const PhpFilePathsPlugin = require( '@wordpress/scripts/plugins/php-file-paths-plugin' );
[line 26] const RtlCssPlugin = require( '@wordpress/scripts/plugins/rtlcss-webpack-plugin' );
...
[line 39] ...} = require( '@wordpress/scripts/utils' );

```

Next, locate the cssLoaders variable, and update the plugins value inside the postcss-loader (starts line 76):

```
  plugins: [
            require('autoprefixer')({ grid: true }),
            require('@tailwindcss/postcss'),
            ...(isProduction ? [
              require('cssnano')({
                preset: [
                  'default',
                  {
                    discardComments: {
                      removeAll: true,
                    },
                  },
                ],
              }),
            ] : []),
          ],
```

## Step 4 - Convert sass files to css files, import tailwindcss

Tailwind V4 recommends against using preprocessors alongside it. By default, create-block generates sass files, so we just have to convert them to .css files, and update the imports.

In the /src/[block-name] folder, change editor.scss and style.scss to editor.css and style.css. inside edit.js and index.js, update the the editor.scss and style.scss imports to .css.

Finally, import tailwind in editor.css and style.css. You may want to load sources manually so that tailwind doesnt look through every file in the project, which can be done like so:

```
@import "tailwindcss" source(none);
@source "/edit.js";
@source "./save.js";
```

`@import tailwindcss" source(none);` tells tailwind not to check any files. then the `@source` values explicitly target edit.js and save.js. You can read more about this [here](https://tailwindcss.com/docs/detecting-classes-in-source-files#explicitly-registering-sources).

## Step 5 - Build something!

run `npm run start` and you should be off to the races!