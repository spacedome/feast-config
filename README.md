# FEAST Configurator

---

This is a webapp for configuring calls to the [FEAST Eigenvalue Solver](http://www.ecs.umass.edu/~polizzi/feast/index.htm).
It gives an interface for setting input parameters.

This is currently a proof of concept, but covers the more common features of FEAST version 3.0.


---

## Development Install Instructions

Install the dependencies (requires npm).

```bash
cd feast-config
npm install
```

Start dev run (uses [Rollup](https://rollupjs.org)).

```bash
npm run dev
```

Navigate to [localhost:5000](http://localhost:5000).

By default, the server will only respond to requests from localhost. To allow connections from other computers, edit the `sirv` commands in package.json to include the option `--host 0.0.0.0`.

## Building for Production

To create an optimised version of the app:

```bash
npm run build
```


## Deploying to the web

Drop the `/public` folder into a web server.
