# Svelte Tutorial

Below are a few simple tutorials for building interactive web applications using the SvelteKit framework.

### Resources
- [Fire Svelte](https://github.com/blazed-space/fire-svelte)
  - SvelteKit framework featuring:
    - [TailwindCSS](https://tailwindcss.com/)
    - [HTML Monolith Boilerplate](https://github.com/tyler-ruff/tyler-ruff/blob/main/Web-Gallery/HTML-Snippets/index.html)
    - Blazed [custom-scrollbar](https://github.com/tyler-ruff/tyler-ruff/blob/main/Web-Gallery/CSS-Snippets/custom-scrollbar.css)
    - [blink-stop](https://github.com/tyler-ruff/tyler-ruff/blob/main/Web-Gallery/CSS-Snippets/blink-stop.css)
    - Netlify.toml (for easy [Netlify](https://netlify.com/) deploy)
    - Mobile Burger Menu
- [Svelte Documentation](https://svelte.dev/docs)
- [Getting Started Tutorial](https://svelte.dev/tutorial/basics)
- [SvelteKit Documentation](https://kit.svelte.dev/docs/introduction)

## Insert Data into \<head>
```html
    <svelte:head>
        <title>Welcome</title>
    </svelte:head>

    <h1>Hello and welcome to my site!</h1>

    <a href="/about">About my site</a>
```

## Dynamic Parameters

> Dynamic parameters are encoded using [brackets]. For example, a blog post might be defined by src/routes/blog/[slug].svelte. These parameters can be accessed in a load function or via the page store.
- **SvelteKit Docs**
> A route can have multiple dynamic parameters, for example src/routes/[category]/[item].svelte or even src/routes/[category]-[item].svelte. (Parameters are 'non-greedy'; in an ambiguous case like x-y-z, category would be x and item would be y-z.)
- **SvelteKit Docs**

An example can be seen below.

1. src/routes/items/[id].js:

```js
    import db from '$lib/database';
    /** @type {import('./__types/[id]').RequestHandler} */
    export async function get({ params }) {
        // `params.id` comes from [id].js
        const item = await db.get(params.id);
        
        if (item) {
            return {
            body: { item }
            };
        }
        
        return {
            status: 404
        };
    }
```

> The job of a request handler is to return a { status, headers, body } object representing the response, where status is an HTTP status code:
* 2xx — successful response (default is 200)
* 3xx — redirection (should be accompanied by a location header)
* 4xx — client error
* 5xx — server error
- **SvelteKit Docs**

2. src/routes/items/[id].svelte:
```html
    <script>
        // populated with data from the endpoint
        export let item;
    </script>

    <h1>{item.title}</h1>
```

### Sources
- [SvelteKit Docs](https://kit.svelte.dev/docs/routing#pages)

## POST, PUT, PATCH, DELETE
> Endpoints can handle any HTTP method — not just GET — by exporting the corresponding function:
```js
    export function post(event) {...}
    export function put(event) {...}
    export function patch(event) {...}
    export function del(event) {...} // `delete` is a reserved word
```
- **SvelteKit Docs**

> These functions can, like get, return a body that will be passed to the page as props. Whereas 4xx/5xx responses from get will result in an error page rendering, similar responses to non-GET requests do not, allowing you to do things like render form validation errors:
1. src/routes/items.js:
```js
    import * as db from '$lib/database';
    
    /** @type {import('./__types/items').RequestHandler} */
    export async function get() {
        const items = await db.list();
        
        return {
            body: { items }
        };
    }
    
    /** @type {import('./__types/items').RequestHandler} */
    export async function post({ request }) {
        const [errors, item] = await db.create(request);
        
        if (errors) {
            // return validation errors
            return {
            status: 400,
            body: { errors }
            };
        }
        
        // redirect to the newly created item
        return {
            status: 303,
            headers: {
            location: `/items/${item.id}`
            }
        };
    }
```
2. src/routes/items.svelte:
```html
    <script>
        // The page always has access to props from `get`...
        export let items;
        // ...plus props from `post` when the page is rendered
        // in response to a POST request, for example after
        // submitting the form below
        export let errors;
    </script>

    {#each items as item}
        <Preview item={item}/>
    {/each}

    <form method="post">
        <input name="title">

        {#if errors?.title}
            <p class="error">{errors.title}</p>
        {/if}

        <button type="submit">Create item</button>
    </form>
```
- **SvelteKit Docs**

### Sources
- [SvelteKit Docs](https://kit.svelte.dev/docs/routing#endpoints-post-put-patch-delete)

## Accept Input Property

* Slot (Required)
```html
    <!-- Widget.svelte -->
    <div>
        <slot name="header">No header was provided</slot>
        <p>Some content between header and footer</p>
        <slot name="footer"></slot>
    </div>

    <!-- App.svelte -->
    <Widget>
        <h1 slot="header">Hello</h1>
        <p slot="footer">Copyright (c) 2019 Svelte Industries</p>
    </Widget>
```

* \$$slots (Conditional)
```html
    <!-- Card.svelte -->
    <div>
        <slot name="title"></slot>
        {#if $$slots.description}
            <!-- This <hr> and slot will render only if a slot named "description" is provided. -->
            <hr>
            <slot name="description"></slot>
        {/if}
    </div>

    <!-- App.svelte -->
    <Card>
        <h1 slot="title">Blog Post Title</h1>
        <!-- No slot named "description" was provided so the optional slot will not be rendered. -->
    </Card>
```

### Sources
- [Svelte Docs](https://svelte.dev/docs#template-syntax-slot-slot-name-name)

## Accept Query Parameter
* Requires SvelteKit
```html
    <script>
        import { page } from '$app/stores'
        const email = $page.url.searchParams('email')
    </script>
```

### Sources
- [Accept Query Parameters](https://stackoverflow.com/questions/66637632/access-url-query-string-in-svelte)
  
## Use Firebase in SvelteKit
1. Install Firebase and create config files
```sh
firebase init
```
2. Import Firebase & Firestore, Config, and Query DB
```html
    <script>
        import "firebase/app";
        import { getFirestore, collection, getDocs } from 'firebase/firestore/lite';
        import { initializeApp } from 'firebase/app';
            const firebaseConfig = {
            apiKey: "{API KEY}",
            authDomain: "{AUTH DOMAIN}",
            projectId: "{PROJECT ID}",
            storageBucket: "{STORAGE BUCKET}",
            messagingSenderId: "{SENDER ID}",
            appId: "{APP ID}",
            measurementId: "{MEASUREMENT ID}"
        };
        const app = initializeApp(firebaseConfig);
        // Write Functions for each Query
            async function getProjects() {
            const db = getFirestore(app);
            const projectsCol = collection(db, '{COLLECTION}');
            const projSnapshot = await getDocs(projectsCol);
            const projectList = projSnapshot.docs.map(doc => doc.data());
            return projectList;
        }
        let limit = 4;
        let index = 0;
        let moreButton = false;
        /**
            * @type {import("firebase/firestore/lite").DocumentData[]}
        */
        let projectsFull = [];
        /**
            * @type {import("firebase/firestore/lite").DocumentData[]}
        */
        let projects = [];
        // Bind Query Function On Mount
        onMount(async () => {
            projectsFull = await getProjects();
            projects = projectsFull.slice(0, limit);
            index += limit;
            if(index != projectsFull.length){
                moreButton = true;
            }
        });
        // Load More Items
        function loadMore(){
            index += limit;
            projects = projectsFull.slice(0, index);
            if(index == projectsFull.length){
                moreButton = false;
            }
        }
    </script>
    <section>
        <div>
            <h1>
                Our Projects
            </h1>
            <!-- 
                On mount function will asyncronously load elements, which will be iterated over,
                exiting as an "array" object on the "projects" variable.
            -->
            <div>
                {#each projects as project}
                    <a href="{project.url}">
                        <div>
                            <img src="{project.image}?w=80&h=80" alt="">
                            <h1>
                                {project.name}
                            </h1>
                            <p>
                                {project.version}
                            </p>
                        </div>
                    </a>
                {:else}
                    <div>
                        <!-- Empty State -->
                    </div>
                {/each}
            </div>
            <div>
                {#if moreButton == true}
                    <button on:click={loadMore}>
                        Load More
                    </button>
                {:else}
                    <span class="hidden">
                        All content loaded.
                    </span>
                {/if}
            </div>
        </div>
    </section>
```

### Sources
- [Building CRUD Apps with Firebase and Svelte](https://blog.logrocket.com/building-crud-application-svelte-firebase/)
- [Firebase Docs](https://firebase.google.com/docs/build)
- [Svelte Docs](https://svelte.dev/docs)

## Use Sanity with Svelte
1. Install Sanity Client
```sh
npm i @sanity/client
```
2. Create sanityClient.js
```js
    // sanityClient.js
    import sanityClient from "@sanity/client";

    const client = sanityClient({
    projectId: "YOUR_PROJECT_ID",
    dataset: "YOUR_PROJECT_DATASET",
    apiVersion: "2022-03-24", // choose the API version you want
    useCdn: true,
    });

    export default client;
```
3. Query Sanity (ex. src/routes/index.svelte)
```html
    <script>
        import client from "../sanityClient";

        export const get = async () => {
            const genuses = await client.fetch(/* groq */ `*[
                _type == "genus"
            ][0..10]{
                _id,
                name,
            }`);

            return {
                body: {
                    genuses,
                },
            };
        };
    </script>
```
4. Example using wildcard route (routes/genuses/[_id].js)

```js
    import client from "../../sanityClient";

    export const get = async (event) => {
        // Access the _id from the params object
        const { _id } = event.params;
        const genus = await client.fetch(
            /* groq */ `*[
            _type == "genus" &&
            _id == $_id
            ][0]`,
            { _id }
        );

        if (!genus?._id) {
            return {
            status: 404,
            };
        }

        return {
            body: {
            genus,
            },
        };
    };
```

```html
    <script>
        export let genuses = [];
    </script>

    <ul>
        {#each genuses as genus}
            <li>
                <a href={`/genuses/${genus._id}/`}>{genus.name}</a>
            </li>
        {/each}
    </ul>
```

### Sources
- [Using Sanity in SvelteKit](https://hdoro.dev/sanity-io-to-svelte-kit)