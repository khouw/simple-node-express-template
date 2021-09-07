# NodeJS - Express Template

Basic NodeJS-Express Boilerplate from [here](https://javascript.plainenglish.io/a-minimal-node-js-express-babel-setup-part-1-6be7b3f3bb55 "here") which includes:
- [NodeJS](https://nodejs.org/en/ "NodeJS")
- [Express](https://expressjs.com/ "Express")
- [Babel](https://babeljs.io/ "Babel")
- [ESLint](https://eslint.org/ "ESLint")
- [Prettier](https://prettier.io/ "Prettier")
- [Nodemon](https://nodemon.io/ "Nodemon")
-----
If you clone this repository, you're good to go. But if you want to create this template on your own, follow these step by step instructions.
## Step by Step Instructions.
### 1. Node and Express Setup
Create a new project with the [Express application generator](https://expressjs.com/en/starter/generator.html "Express application generator").
```
mkdir node-express-template
cd node-express-template
npx express-generator .
```

This tool generates a simple Express framework. Run the initial `npm install` to install the packages, then run `npm start` script that the Express generator put in `package.json`. The server should be up on http://localhost:3000.

Create a `src` directory at the root of our project and move these things into it: `bin`, `public`, `routes`, `views`, and `app.js`. Then, change the file `www` (in `src/bin`) to `www.js`.


### 2. Babel Setup
Babel will "transpile" our ES2015+ code and module syntax to older-style code for compatibility purposes.

    npm install --save-dev @babel/core @babel/cli @babel/preset-env @babel/node

- `@babel/core` is the Babel core compiler
- `@babel/cli` is Babel command line tools
- `@babel/preset-env` is one of the official presets available through Babel.
- `@babel/node` works just like the Node cli itself, but of course runs the Babel process, too. So, instead of running e.g. `node index.js` to run the Node process, you can use `babel-node index.js`.

Create a Babel config file at the root level of your project called `.babelrc.json`, and give it the following contents:

    {
      "presets": [
        "@babel/preset-env"
      ]
    }

With our core Babel packages installed and `.babelrc.json` set up, let's update our npm scripts. In the scripts section of `package.json`, remove the `start` command that Express generator made for us, and add these new ones:

    // ... other parts  
    "scripts": {
        "dev": "babel-node ./src/bin/www.js",
        "clean": "rm -rf dist",
        "build": "npm run clean && babel ./src --out-dir dist --copy-files",
        "prod": "node ./dist/bin/www.js",
      }
    // ... other parts

- `dev` — using our `@babel/node` package we installed, this is an easy way to do local development. Just like using `node`, but takes care of Babel tranpilation for us.
- `clean` — the build command (next) outputs the result of the Babel build in a `dist` folder… this simply deletes that built directory so we can start fresh each time.
- `build` — run the babel process on our source files so that we have a `dist` directory containing our transpiled code, ready to be run in production with normal `node`.
- `prod` — assuming we've built our code with the build command, we can now run it with `node`.

Run `npm run dev` and everything should work just like before.

#### 4a. Polyfill
We set up our build process above to leverage Babel with its `preset-env` to transform syntax of ES2015+ code back to older-style syntax so that it runs across more environments. An example of transforming syntax is converting an arrow function `const x = () => {}` to a normal function `function x() {}`. However, the concept of a polyfill is slightly different. A polyfill is a piece of code that actually uses primitives of an older target version of the language to add features the language so it's compatible with our newer code. For example, the fetch call we often use in web development. There's no concept of transforming syntax from ES2015+ fetch to something older, but rather a polyfill is written to add a compatible fetch call.

So, to make sure that the correct things are polyfilled for us, let's install these libraries:

    npm install --save core-js regenerator-runtime

Then, add these as the first 2 lines in src/bin/www (after `#!/user/bin/env node`):

    import 'core-js/stable';
    import 'regenerator-runtime/runtime';


#### 4b. Imports resolver
Sometimes, writing imports in your code can be like `../../../directoryX/thing`. With import resolver, we can create names and "link" it to any directory we want. It simplifies writing the directory import. This is easy to do using a Babel plugin.

Install the plugin with:

    npm install --save-dev babel-plugin-module-resolver

Then, update `.babelrc.json` to look like this:

    {
      "presets": [
        "@babel/preset-env"
      ],
      "plugins": [
        ["module-resolver", {
          "alias": {
            "#routes": "./src/routes",
          }
        }]
      ]
    }

You'll see we added a new `plugins` section, where we use our new plugin. Most importantly, see the `alias` object. This is where we can make up whatever names we'd like as aliases to use in our import statements throughout our code. As a sample, you see that `#routes` is now an alias for our routes directory under src. The # character isn't required, but I've seen others use it as an easy way to see in your code that you're using a custom alias.

With our new alias, head back to `src/app.js` file. We have two imports in here for our routes:

    import indexRouter from './routes/index';
    import usersRouter from './routes/users';

These imports are very straightforward so you wouldn't necessarily need/want to use the aliases here, but let’s do it for the example nonetheless. Now they’ll look like this (notice no leading dot or slash):

    import indexRouter from '#routes/index';
    import usersRouter from '#routes/users';

Restart your Node server and things will work just as before. Notice this is just a dev dependency — when you run `npm run build` and look at `dist/app.js`, you’ll see that Babel changes those absolute imports back to relative require statements.

### 3. Update Code to Modern Syntax

The main updates we'll be making in our boilerplate files:
- Convert to ES Modules (`export`, `export default` and `import` syntax rather than Common JS `module.exports` and `require` syntax)
- Switch to `const` variables (block-scoped) instead of `var` variables
- Use arrow functions

This is the updated `www.js` file
```javascript
#!/user/bin/env node
import 'core-js/stable';
import 'regenerator-runtime/runtime';

/**
 * Module dependencies.
 */

import http from 'http';
import app from '../app';

/**
 * Normalize a port into a number, string, or false.
 */
const normalizePort = (val) => {
  const port = parseInt(val, 10);

  if (Number.isNaN(port)) {
    // named pipe
    return val;
  }

  if (port >= 0) {
    // port number
    return port;
  }

  return false;
};

/**
 * Get port from environment and store in Express.
 */

const port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

/**
 * Create HTTP server.
 */

const server = http.createServer(app);

/**
 * Event listener for HTTP server "error" event.
 */
const onError = (error) => {
  if (error.syscall !== 'listen') {
    throw error;
  }

  const bind = typeof port === 'string' ? `Pipe ${port}` : `Port ${port}`;

  // handle specific listen errors with friendly messages
  switch (error.code) {
    case 'EACCES':
      console.error(`${bind} requires elevated privileges`);
      process.exit(1);
      break;
    case 'EADDRINUSE':
      console.error(`${bind} is already in use`);
      process.exit(1);
      break;
    default:
      throw error;
  }
};

/**
 * Event listener for HTTP server "listening" event.
 */
const onListening = () => {
  const addr = server.address();
  const bind = typeof addr === 'string' ? `pipe ${addr}` : `port ${addr.port}`;
  console.log(`Listening on ${bind}`);
};

/**
 * Listen on provided port, on all network interfaces.
 */
server.listen(port);
server.on('error', onError);
server.on('listening', onListening);
```

This is the updated `app.js` file:
```javascript
import createError from 'http-errors';
import express from 'express';
import path from 'path';
import cookieParser from 'cookie-parser';
import logger from 'morgan';

import indexRouter from './routes/index';
import usersRouter from './routes/users';

const app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use('/', indexRouter);
app.use('/users', usersRouter);

// catch 404 and forward to error handler
app.use((req, res, next) => {
  next(createError(404));
});

// error handler
app.use((err, req, res) => {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

export default app;
```

This is the updated `routes/index.js` file:
```javascript
import express from 'express';
const router = express.Router();

/* GET home page. */
router.get('/', (req, res, next) => {
  res.render('index', { title: 'Express' });
});

export default router;
```

And this is the updated `routes/users.js` file:
```javascript
import express from 'express';
const router = express.Router();

/* GET users listing. */
router.get('/', (req, res, next) => {
  res.send('respond with a resource');
});

export default router;
```

### ESLint and Prettier
Directly from [their site](https://eslint.org/ "their site"), "ESLint statically analyzes your code to quickly find problems". ESLint will check your code to see if you're violating any of its rules (there are many defaults, and you can customize the rule set, too). For example, here's a rule called no-dupe-keys. This one is pretty straightforward — ESLint will consider it an error in your code if you create an object like `const x = { hello: 'world', hello: 'planet' }`. Rules like this are super helpful because they might prevent you from making a mistake. Other rules are less about obvious bugs and more about best practices.

[As they mention](https://prettier.io/ "As they mention"), Prettier is “an opinionated code formatter.” That means they don’t want to give you a ton of options, but would rather you buy in to the code style they help you maintain. We will go over how to make some basic style overrides, though. I don’t mind its strong opinions — the styling is largely accepted by the developer community, and having the automatic formatting is a big win for consistency and ease of development.

Essentially, Prettier takes care of all style-related rules (spacing, commas, semicolons, parentheses in arrow functions), and ESLint takes care of everything not style-related, like issues with unused variables, duplicate keys in object literals, and many more.

### 4. ESLint Setup

    npm install --save-dev eslint eslint-config-airbnb-base eslint-plugin-import


- `eslint` is the main ESLint package
- `eslint-config-airbnb-base`: ESLint runs off of various rule sets that you or a 3rd party can define, which consists of making choices regarding which syntax rules are errors, or simply warnings, etc. The Airbnb rule set is a really popular options to automatically get a solid set of rules going
- `eslint-plugin-import` is a package that helps with linting regarding ES Module style import / export statements along with file paths, etc. In fact, our Airbnb package notes that we need this package as well for use with its rules

Create a file called `.eslintrc.js`, which contains ESLint configuration. For starters, it’ll look like this:

    module.exports = {
      env: {
        es2021: true,
      },
      extends: ['airbnb-base'],
      parserOptions: {
        ecmaVersion: 12,
        sourceType: 'module',
      },
    };

1. We specify the environment, which defines a set of global variables. We use the modern `es2021`.
2. We extend `airbnb-base` — now we’re using the Airbnb linting rules.
3. We set the `ecmaVersion` and `sourceType` in our `parserOptions` — the `ecmaVersion` is actually set up automatically by our `es2021` `env` value (es2021 === es12), but we’ll leave it in here, and we’re using ES Modules, hence module.

Add this line as a new script in `package.json`:

    "scripts": {
      // ...other scripts,
      "lint": "eslint \"src/**/*.js\""
    }

Run `npm run lint` from the root of your project and see what pops up.

### 5. Prettier Setup
To install prettier, run this command:

    npm install --save-dev prettier

Then add more script to `package.json`:

    "scripts": {
      // ... other scripts,
      "format": "prettier --write \"src/**/*.js\""
    }

Run `npm run format`, which will tell Prettier to automatically go through and format the code. We can customize with a` .prettierrc` file in the root of your project, for example:

    {
      "semi": true,
      "trailingComma": "all",
      "singleQuote": true,
      "printWidth": 80
    }

At the moment, there will be a bunch of cases where ESLint and Prettier rules conflict, making it impossible to satisfy both. For example, you might run a Prettier format, which will change the code in such a way creates ESLint /Airbnb errors… but you can’t change it back because Prettier will then report errors.

Let’s get them working together. Add these 2 packages:

    npm install --save-dev eslint-plugin-prettier eslint-config-prettier

- `eslint-plugin-prettier` is an ESLint plugin that runs Prettier in the background and reports results as ESLint errors. We configure this in the .eslintrc.js file, since it’s an ESLint plugin, and the result is that ESLint will be steering the ship, using Prettier as a tool in the background
- `eslint-config-prettier` is an ESLint configuration which simply turns off any rules that would conflict with Prettier style. As they mention, since they simply turn off rules and don’t do anything else, you want to use this in tandem with another rule set (i.e. our Airbnb rule set), turning off rules that’ll conflict.

Update your `.eslintrc.js` to look like this, now:

    module.exports = {
      env: {
        es2021: true,
      },
      extends: ['airbnb-base', 'prettier'],
      parserOptions: {
        ecmaVersion: 12,
        sourceType: 'module',
      },
      rules: {
        'prettier/prettier': 'error',
      },
      plugins: ['prettier'],
    };

- In the `extends` key, we keep `airbnb-base`, but add in `prettier` after the Airbnb rules, which says “use the Airbnb rules, but then beyond that, disable whatever rules are specified in `eslint-config-prettier`"
- In `rules`, we’re marking issues that show up via Prettier (in `eslint-plugin-prettier`) as ESLint errors.
- In `plugins`, we have to specify that we are indeed using `eslint-plugin-prettier`

ESLint now has all rules that might conflict with Prettier turned off, so we know that when we run Prettier (`npm run format` as we have it set up), it can no longer result in formatted code that ESLint will consider an error.

Earlier in part 4b, we had added a Babel plugin called `babel-plugin-module-resolver` to help us define aliases to enable imports resolver in our code. Here’s an import resulting from that setup:

    import indexRouter from '#routes/index';

Since Babel takes care of these aliases in the build process, it’s no surprise that ESLint doesn’t understand what we’re referring to, here. Thankfully, we can use `eslint-import-resolver-babel-module`. This helps ESLint understand that it can look on the Babel alias paths to confirm the imported files exist.

    npm install --save-dev eslint-import-resolver-babel-module

And then we’ll add a settings key to our `.eslintrc.js` file just like the plugin specifies in its docs.

    // ... the rest of your .eslintrc.js
      settings: {
        'import/resolver': {
          'babel-module': {},
        },
      },

With that in place, you can run `npm run lint` once more, and you’ll see that category of errors disappear.

### 6. Nodemon Setup
To install nodemon, run this command:

    npm install nodemon --save-dev

Then, head to `package.json` and modify `dev` script to become like this:

    // ... the rest of your package.json
    "scripts": {
            "dev": "nodemon --exec babel-node ./src/bin/www.js",
            // ... the rest of your scripts
    
        },

### VS Code Extensions (optional)
Everything before this is all still good whether or not you use VS Code. This section is just a quick rundown of how you can make this process even smoother. Rather than running the npm commands for linting and formatting, we’ll add VS Code plugins that constantly watch over our code.

You’ll install these 2 VS Code extensions:
- ESLint (by Dirk Baeumer)
- Prettier (by Prettier)