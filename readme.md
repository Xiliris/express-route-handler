# Express Route Handler

Dynamic Express Route Handler is a utility for Express.js applications that automatically loads routes from a specified directory.

## Features

- Automatically loads all `.js` files in a specified directory and its subdirectories as routes.
- Supports nested routes based on the directory structure.
- Provides logging functionality to track the loading process.
- Allows the application of middleware to specific routes for enhanced functionality and control.
- Ignores files starting with an underscore and directories surrounded by parentheses.

## Installation

```bash
npm install @xiliris/express-route-handler
```

## Contact

```arduino
https://adnanskopljak.com
```

### Usage:

```js
const express = require("express");
const RouteHandler = require("@xiliris/express-route-handler");
const path = require("path");

const app = express();
const routesDirectory = path.join(__dirname, "routes");
const data = {
  log: true,
};

const routeHandler = new RouteHandler(app, routesDirectory, data);
routeHandler.handleRoutes();

app.listen(3000, () => {
  console.log("Server is running on port 3000");
});
```

## Explanation

### new RouteHandler(app, dir, data)

- app: An instance of an Express application.
- dir: The path to the directory containing the route files.
- data: An object with the following properties:
  > log: A boolean indicating whether to log the loading process.

### Middleware Configuration

- The middleware property in the data object allows you to specify middleware to be applied to certain routes. Each middleware configuration object should have the following properties:

- path: The route path where the middleware should be applied.
- middlewares: An array of middleware functions to be applied.

### File Structure

```
.
├── routes                     # Directory where route files are stored
│   ├── index.js               # Treated as "/"
│   ├── users                  # Treated as a nest, "/users"
│   │   ├── index.js           # Treated as "/users"
│   │   └── [id]               # Treated as a nest, "/users/:id"
│   │       └── index.js       # Treated as "/users/:id"
│   ├── products               # Treated as a nest, "/products"
│   │   ├── index.js           # Treated as "/products"
│   │   ├── [id]               # Treated as a nest, "/products/:id"
│   │   │   └── index.js       # Treated as "/products/:id"
│   │   ├── _archive.js        # Ignored because it starts with an underscore
│   │   └── (old)              # Ignored because it's surrounded by parentheses
│   │       └── old_product.js # Ignored because its parent directory is ignored
│   └── orders                 # Treated as a nest, "/orders"
│       ├── index.js           # Treated as "/orders"
│       └── [id]               # Treated as a nest, "/orders/:id"
│           └── index.js       # Treated as "/orders/:id"
├── app.js                     # Main file of your Express application

```

- **routes** is the directory where your route files are stored. It contains **index.js**, **users.js**, **products**, and **orders**. The **products** directory contains its own **index.js**, **\_archive.js**, and **(old)** directories. The **(old)** directory is ignored by the route handler because it's surrounded by parentheses. The **\_archive.js** file is also ignored because it starts with an underscore.
- **app.js** is the main file of your Express application. This is where you would use the **routeHandler** function.
- **routes/products/[id]/index.js**, and **routes/orders/[orderId]/index.js** are dynamic routes. The **[id]** part of the folder name is a placeholder that will be replaced with the actual ID in the URL. For example, if you have a URL like **/orders/123**, it will be handled by the **routes/orders/[id]/index.js** route.

## NOTE

- When using dynamic routes (like `[id]`), it's important to create your router with the `{ mergeParams: true }` option. This ensures that the router has access to the parameters of its parent router. Here's how you can do it:

```js
const express = require("express");
const router = express.Router({ mergeParams: true });
```

- Without **{ mergeParams: true }**, **req.params** in the route handler would not contain the **id** parameter from the parent router's path.

### Example with Middleware

```js
const express = require("express");
const path = require("path");
const RouteHandler = require("@xiliris/express-route-handler");
const isMiddleware = require("./middleware/isMiddleware");
const addon = require("./middleware/addon");

const app = express();
const port = 3000;
const routesDirectory = path.join(__dirname, "routes");
const data = {
  log: true,
  middleware: [
    {
      path: "/users",
      middlewares: [isMiddleware, addon],
    },
  ],
};

const routeHandler = new RouteHandler(app, routesDirectory, data);
routeHandler.handleRoutes();

app.listen(port, () => {
  console.log(`Server listening at http://localhost:${port}`);
});
```
