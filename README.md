# React Fetcher

[![Travis][build-badge]][build]
[![Coveralls][coveralls-badge]][coveralls]

Use native fetch via react hooks for controlling RESTful API like [Apollo](https://www.apollographql.com/)

---

## Outline

- [Features](#Features)
- [Installation](#Installation)
- [Schema](#Schema)
- [Usage](#usage)
  - [FetcherClient](#FetcherClient)
  - [Components](#Components)
    - [FetcherProvider](#FetcherProvider)
    - [Fetcher](#Fetcher)
  - [Hooks](#Hooks)
    - [useFetcher](#useFetcher)
    - [useParallelFetcher](#useParallelFetcher)
  - [Server Side Rendering](#Server-Side-Rendering)

## Features

- ✅ Support multiple concurrent requests
- ✅ Support server side rendering
- ✅ Support both render props and hooks usage
- ✅ Zero dependencies
- ✅ Customizable request and response data
- ✅ Cancel requests

## Installation

```bash
yarn add @arthur/react-fetcher
```

## Schema

### Request

```ts
type Request = {
  // endpoint for api
  url: string;

  // HTTPHeaders
  headers?: HTTPHeaders = {};

  // HTTPMethod
  method?: HTTPMethod = HTTPMethod.GET;

  // payload for http request
  payload?: Object = {};
};
```

### Response

```ts
type Response = {
  data: Object;
  loading: boolean;
  error: Error;
};
```

### FetcherClientOptions

```ts
type FetcherClientOptions = {
  // prefix for url, default: '/',
  baseURL?: string = '/';

  // config for http headers
  headers?: HTTPHeaders = {};

  // state for data coming from server side
  initialState?: string;

  // function for dynamic controlling request header
  transformRequest?: (request: Request): Request;

  // function for transform the format of response data
  transformResponse?: (response: Response): Response;
};
```

### FetcherOptions

```ts
type FetcherOptions = Request | {
  lazy?: boolean = false;

  // function for determining get next page or not
  // if returns an object, it will be merged into payload
  // and continue to fetch util returns null
  shouldFetchNextPage?: (data: Object): Object | null;
  ...
};
```

### FetcherResponse

```ts
type FetcherResponse = Response | {
  doFetch: (payload: Object): Response;
  ...
};
```

## Usage

### FetcherClient

```ts
class FetcherClient {
  constructor(options: FetcherClientOptions) {}
  get(options: Request): Response {}
  post(options: Request): Response {}
  put(options: Request): Response {}
  delete(options: Request): Response {}
}
```

#### Examples

Create a fetcher client

```js
import { FetcherClient } from '@arthur/react-fetcher';

const client = new FetcherClient({
  baseURL,
  headers,
  transformRequest,
  transformResponse
});
```

Get Request

```js
const payload = {};

client.get('/posts', payload).then(response => console.log(response));
```

Post Request

```js
const payload = {
  title: 'Hello, React Fetcher'
};

client.post('/posts', payload).then(response => console.log(response));
```

### Components

#### FetcherProvider

```ts
function FetcherProvider({ client: FetcherClient }): ReactElement;
```

##### Examples

Connect client to react

```js
import React from 'react';
import { render } from 'react-dom';
import { FetcherClient, FetcherProvider } from '@arthur/react-fetcher';

const client = new FetcherClient();
const container = document.getElementById('root');

const App = () => (
  <FetcherProvider client={client}>
    <h2>Hello, React Fetcher</h2>
  </FetcherProvider>
);

render(<App />, container);
```

#### Fetcher

```ts
function Fetcher(props: FetcherOptions | {
  children: (props: FetcherResponse): ReactElement
}): ReactElement {}
```

##### Examples

```js
import { Fetcher } from '@arthur/react-fetcher';

const payload = { first: 10 };

const Posts = () => (
  <Fetcher method="GET" url="/posts" payload={payload}>
    {(error, loading, data) => {
      if (loading) {
        return <div>Loading...</div>;
      } else if (error) {
        return <div>Oops...something wrong: ${error.toString()}</div>;
      }

      return (
        <div>
          {data.map(({ id, title }) => (
            <div key={id}>
              <h2>{title}</h2>
            </div>
          ))}
        </div>
      );
    }}
  </Fetcher>
);
```

### Hooks

#### useFetcher

```ts
function useFetcher(options: FetcherOptions): FetcherResponse {}
```

##### Examples

```js
import { useFetcher } from '@arthur/react-fetcher';

const Posts = () => {
  const { loading, error, data } = useFetcher({
    method: 'GET',
    url: '/posts',
    payload: {
      first: 10
    }
  });

  if (loading) {
    return <div>Loading...</div>;
  } else if (error) {
    return <div>Oops...something wrong: ${error.toString()}</div>;
  }

  return (
    <div>
      {data.map(({ id, title }) => (
        <div key={id}>
          <h2>{title}</h2>
        </div>
      ))}
    </div>
  );
};
```

#### useParallelFetcher

```ts
function useParallelFetcher(options: {
  [key: string]: FetcherOptions;
}): FetcherResponse {}
```

##### Examples

```js
import { useParallelFetcher } from '@arthur/react-fetcher';

const Posts = () => {
  const { loading, error, data } = useParallelFetcher({
    posts: {
      url: '/posts'
    },
    users: {
      url: '/users'
    }
  });

  if (loading) {
    return <div>Loading...</div>;
  } else if (error) {
    return <div>Oops...something wrong: ${error.toString()}</div>;
  }

  return (
    <div>
      {data.posts.map(({ id, title }) => (
        <div key={id}>
          <h2>{title}</h2>
        </div>
      ))}
      {data.users.map(({ id, name }) => (
        <div key={id}>
          <h2>{name}</h2>
        </div>
      ))}
    </div>
  );
};
```

### Server Side Rendering

```ts
async function getFetcherState(
  App: ReactElement,
  client: FetcherClient
): string {}
```

#### Examples

```js
// app.js
import express from 'express';
import { renderToStaticMarkup } from 'react-dom';
import render from './render';

const app = new Express();
const host = process.env.HOST || 'localhost';
const port = process.env.PORT || 3000;

app.use(async (req, res) => {
  const html = await render();

  res.status(200);
  res.send(`<!doctype html>\n${renderToStaticMarkup(html)}`);
  res.end();
});

app.listen(port, host, error => {
  if (error) {
    console.error(error);
    return;
  }

  console.log(`Server started on http://${host}:${port}`);
});

// render.js
import React from 'react';
import { renderToString } from 'react-dom';
import { getFetcherState, FetcherClient } from '@arthur/react-fetcher';
import App from '@/App';

const HTML = ({ content, fetcherState }) => (
  <html>
    <body>
      <div id="root" dangerouslySetInnerHTML={{ __html: content }} />
      <script
        dangerouslySetInnerHTML={{
          __html: `window.__FETCHER_STATE__=${fetcherState};`
        }}
      />
    </body>
  </html>
);

const render = async () => {
  const client = new FetcherClient();
  const SSRApp = (
    <FetcherProvider client={client}>
      <App />
    </FetcherProvider>
  );

  // please make sure getFetcherState is running before renderToString
  const fetcherState = await getFetcherState(SSRApp, client);
  const content = renderToString(SSRApp);

  return <HTML content={content} fetcherState={fetcherState} />;
};

export default render;
```

## License

MIT

[build-badge]: https://img.shields.io/travis/arthur791004/react-fetcher/master.png?style=flat-square
[build]: https://travis-ci.org/arthur791004/react-fetcher
[coveralls-badge]: https://img.shields.io/coveralls/arthur791004/react-fetcher/master.png?style=flat-square
[coveralls]: https://coveralls.io/github/arthur791004/react-fetcher
