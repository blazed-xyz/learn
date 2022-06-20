# React Tutorials

Below are a few simple tutorials for building interactive web applications using the React framework.

### Resources
- [Fire React](https://github.com/blazed-space/fire-react)
  - React framework featuring:
    - [TailwindCSS](https://tailwindcss.com/)
    - [HTML Monolith Boilerplate](https://github.com/tyler-ruff/tyler-ruff/blob/main/Web-Gallery/HTML-Snippets/index.html)
    - Blazed [custom-scrollbar](https://github.com/tyler-ruff/tyler-ruff/blob/main/Web-Gallery/CSS-Snippets/custom-scrollbar.css)
    - [blink-stop](https://github.com/tyler-ruff/tyler-ruff/blob/main/Web-Gallery/CSS-Snippets/blink-stop.css)
    - Netlify.toml (for easy [Netlify](https://netlify.com/) deploy)
- [React Documentation](https://reactjs.org/docs/getting-started.html)
- [Getting Started Tutorial](https://reactjs.org/tutorial/tutorial.html)
- [Create React App Documentation](https://create-react-app.dev/)

## Setup Client-Based Routing
1. First, install React Router v6
```sh
npm install react-router-dom@6
```
2. Then, open src/index.js, and insert the following:
```js
    import * as React from "react";
    import ReactDOM from "react-dom/client";
    import { BrowserRouter } from "react-router-dom";
    import "./index.css";
    import App from "./App";
    import reportWebVitals from "./reportWebVitals";

    const root = ReactDOM.createRoot(
    document.getElementById("root")
    );
    root.render(
    <React.StrictMode>
        <BrowserRouter>
        <App />
        </BrowserRouter>
    </React.StrictMode>
    );
```
3. Open src/App.js and add the following:
```js
    import * as React from "react";
    import { Routes, Route, Link } from "react-router-dom";
    import "./App.css";

    function App() {
        return (
            <div className="App">
                <h1>Welcome to React Router!</h1>
                <Routes>
                    <Route path="/" element={<Home />} />
                    <Route path="about" element={<About />} />
                </Routes>
            </div>
        );
    }
```
4. Further down in src/App.js, create your route pages:
```js
    // App.js
    function Home() {
        return (
            <>
                <main>
                    <h2>Welcome to the homepage!</h2>
                    <p>You can do this, I believe in you.</p>
                </main>
                <nav>
                    <Link to="/about">About</Link>
                </nav>
            </>
        );
    }

    function About() {
        return (
            <>
                <main>
                    <h2>Who are we?</h2>
                    <p>
                        That feels like an existential question, don't you
                        think?
                    </p>
                </main>
                <nav>
                    <Link to="/">Home</Link>
                </nav>
            </>
        );
    }
```

### Sources
- [React Router Documentation](https://reactrouter.com/docs/en/v6)

## Get Parameters from URL
* React Router v6, using hooks
```js
    const [searchParams, setSearchParams] = useSearchParams();
    searchParams.get("__firebase_request_key");
```

### Sources
- [Get Parameters from Query String](https://stackoverflow.com/questions/35352638/how-to-get-parameter-value-from-query-string)