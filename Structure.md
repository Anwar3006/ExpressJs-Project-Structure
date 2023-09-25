# A future-proof Node.js express file/folder structure

- How should we structure folders and files in Node.js? I was unhappy and slightly confused by the MVC approach, and wanted to move to a more colocation/feature-based structure. Here's what I came up with.
- This Project is heavily influenced by and mirrored from [codemzy](https://www.codemzy.com/blog/nodejs-file-folder-structure)

## Node.js + Express

- For the server, I'm using Node.js and Express. Express is described as a "fast, unopinionated, minimalist web framework for Node.js".

- Which is great. Unopinionated means you can structure your files however you want. But it's also bad. Because you can structure your files however you want!

- So, before we get into the details, let's take a look at an example. Here's how I'd structure an API for a library, which will have books and allow users to borrow them.

```
📁 server/
└── 📁 api/
  └── 📁 books/
    ├── 📄 books.js
    ├── 📄 books.handlers.js
    ├── 📄 books.routes.js
    └── 📄 index.js
  ├── 📁 borrow/
    ├── 📄 borrow.js
    ├── 📄 borrow.handlers.js
    ├── 📄 borrow.helpers.js
    ├── 📄 borrow.routes.js
    └── 📄 index.js
  ├── 📁 user/
    ├── 📄 index.js
    ├── 📄 user.js
    ├── 📄 user.handlers.js
    ├── 📄 user.helpers.js
    ├── 📄 user.middleware.js
    └── 📄 user.routes.js
├── 📁 middleware/
├── 📁 utils/
├── 📄 routes.js
└── 📄 server.js
```

## What about MVC?!

If you look at most Node.js tutorials, and certainly when I was learning Node.js, you're going to see the MVC (Model, View, Controller) structure.

But in my experience, it usually ends up being more MVCSRM (Model, View, Controller, Services, Routes, Middleware) with a directory for each.

Here's an example of a folder structure in Node.js with MVC\*:

```
📁 server/
├── 📁 controllers/
  ├── 📄 books.controllers.js
  ├── 📄 borrow.controllers.js
  └── 📄 user.controllers.js
├── 📁 routes/
  ├── 📄 books.routes.js
  ├── 📄 borrow.routes.js
  ├── 📄 index.js
  └── 📄 user.routes.js
├── 📁 services/
  ├── 📄 books.services.js
  ├── 📄 borrow.services.js
  └── 📄 user.services.js
├── 📁 helpers/
  ├── 📄 borrow.helpers.js
  └── 📄 user.helpers.js
├── 📁 middleware/
  └── 📄 user.middleware.js
├── 📁 utils/
└── 📄 server.js
```

As new features get added/updated/removed it becomes harder to keep track of all the files.

When the directories are closed, every project looks the same. At first glance, I really don't know what this server does. And I need to open up every directory to find out which ones each feature uses.

For example, let's say I need to add a new endpoint to books. I've got to open the routes, controllers, services and probably helpers too. Then open up all the files relating to books.

You've got to open all those different directories to check which files relate to books. For example, there's no books.helpers.js file, but you wouldn't know until you check the helpers directory.

With this new folder structure, I can head straight to the books directory, and all the files I need are in one place. You instantly know which files relate to books because they are all neatly together in one place.
It's also more future-proof for the inevitable changes (and versions) that will happen throughout the life of your code.

If I want to add a separate file for validation, I just add a books.validation.js file to the books directory. If I want to add some helper functions, I can add books.helpers.js. Any files I need for books, go into the books directory. Simple.

And let's say our library decided to get rid of books, and loan games instead. I can just delete the books directory, instead of fumbling about opening every other folder looking for files with books in the title.

Or if you want to upgrade your server to a totally different framework or code library, you can do it incrementally, one directory at a time. Upgrade books now and borrowing later.

If you need to move your files or rename books to booksV1\*, you'll only need to change one import statement.

If you are struggling with the MVC approach, maybe this type of colocation will work better for you too.

## The structure

Ok, let's take a more detailed look at this alternative folder structure for Node.js and see how it works.

### API

I call this first folder api but if your server is more than an API, and serves HTML too, you could call it app or features.

Here's an example of what's included in the books folder.

```
📁 server/
└── 📁 api/
  └── 📁 books/
    ├── 📄 books.js
    ├── 📄 books.handlers.js
    ├── 📄 books.routes.js
    └── 📄 index.js
```

You can name the files inside books whatever you like. The thing to remember is that any file that only relates to books, belongs in the books directory.

When I need to add a new base route, like api.mylibrary.com/games, I'll add a games folder to the api directory.

The great thing about this pattern is everything for games will be in that new directory. If I need to add a new route for games, I can open up the folder and everything I need is in one place. The games specific routes, middleware and helpers are all used together and stored together.

### Routes

I set up my main route structure in routes.js. This file ties all of the different features of the server together.

For example, routes.js will say any routes starting with /books will be handled by the ./api/books directory.

```javascript
// import the routes for '/books'
// ./api/books/books.routes.js or ./api/books
const bookRoutes = require("./api/books");
// or ES6 module
// import bookRoutes from './api/books';

// wire up to the express app
app.use("/api/books", bookRoutes);
```

Here's how /api/books/index.js would look in common js.

```javascript
// export the book router
module.exports = require("./books.routes");
```

And here's how it looks in ES6.

```javascript
export { default } from "./books.routes";
```

It's one of the benefits of this Node.js folder structure - the routes.js file is the only file you need outside of the directory, so it's the only thing you need to export!

For the route in the books folder:

```javascript
const express = require("express");
const booksRoutes = express.Router();
// or ES6
// import { Router as booksRoutes } from 'express';

// Handlers
const booksHandlers = require("./books.handlers");

// list all books
booksRoutes.get("/", booksHandlers.books);

// export bookRoutes
module.exports = booksRoutes;
// or ES6
// export default booksRoutes;
```

### Handlers/Controllers

You've probably noticed the .handlers.js files and wondered what they are about.

You could also call these functions controllers, but I chose handlers because it’s shorter and I felt it was clearer since I wasn’t strictly following the MVC approach.

```
HANDLER is the function executed when the route is matched.
```

Routes call the handlers, and the handler handles the request. It's responsible for taking a request and calling any functions it needs to send a response.

Here's an example from books.handlers.js.

```javascript
const { listBooks } = require("./books");

exports.books = async function (req, res) {
  const { user } = req.body;
  const filters = req.query;
  try {
    // get list of books
    const result = await listBooks({ user, filters });
    res.json(result);
  } catch (e) {
    console.log(e.message);
    res.status(500).send({ error: "Problem fetching books." });
  }
};
```

### Features

The handler will call functions that contain the business logic, for example, a listBooks function that reaches out to the database to fetch a list of books.

In the MVC pattern, these are called services or workers.

I tend to put these functions in the main books.js file, but the nice thing about this pattern is you can call them whatever you like. You could have each function in a different file, getBooks.js and getBook.js - or keep these functions together in one file.

### Helpers/Utils

Any function used by books that are not used anywhere else can stay in the books directory. I call these helpers e.g. books.helpers.js because they are specific to help that feature or service.

If it's more of a generic function used throughout the services, then I put those functions in the server/utils folder.

### Middleware

The same as utils. If the middleware is specific to a feature, I'll put it in that features directory, e.g. books.middleware.js. But if it's shared with other parts of the server, I'll put it in the server/middleware folder.

Another benefit to this approach is I know what files and functions are only used in one place, and which ones are shared - which is useful when removing routes or features.
