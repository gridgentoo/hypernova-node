# код airbnb : : hypernova-node

Надеемся, опыт компании Airbnb пригодится всем, кто использует Node.js для решения задач серверного рендеринга.

Использование серверного рендеринга на Node.js означает повышенную вычислительную нагрузку на систему. Такая нагрузка отличается от традиционной, связанной, преимущественно, с обработкой операций ввода-вывода. Именно в таких ситуациях платформа Node.js показывает себя наилучшим образом. В нашем случае, мы, попытавшись добиться высокой производительности серверной части приложения, столкнулись с рядом проблем, которые удалось решить, поработав над архитектурой системы и воспользовавшись вспомогательными механизмами, такими, как nginx и HAProxy.

As Airbnb builds more of its Frontend around Server Side Rendering, we took a look at how to optimize our server configurations to support it.  
https://medium.com/airbnb-engineering/operationalizing-node-js-for-server-side-rendering-c5ba718acfc9  

[01.12.2014] Использование Node.js и Redis при построении приложений с высокой степенью масштабируемости https://drive.google.com/drive/folders/1l0_dWRGqlEHWwGyaJFD_0b7NTsIA7K5d  

[24 июля 2018] Node.js для решения задач серверного рендеринга в Airbnb  
https://docs.google.com/document/d/1g5DDrJBhl-9PyNFsX5NVpmAX_uX_9X1V4-SzjM6Hyvs/  

[28 марта 2019] Как мы пилили серверный рендеринг и что из этого вышло  
https://docs.google.com/document/d/1Stgohn-I9WZ6aLFiYSvsHzKZfVEiHecLgShlyjpAlew/  

# hypernova-client

> A node client for sending requests to Hypernova.

## class Renderer

### Renderer.prototype.addPlugin

> (plugin: HypernovaPlugin)

Adds a plugin to the renderer.

### Renderer.prototype.render

> (data: Jobs): Promise<string>

Sends a request to Hypernova for the provided payload and returns a promise which will fulfill
with the HTML string you can pass down to the client.

## Example usage

```js
const express = require('express');
const Renderer = require('hypernova-client');
const devModePlugin = require('../plugins/devModePlugin');

const app = express();

const renderer = new Renderer({
  url: 'http://localhost:3030/batch',
  plugins: [
    devModePlugin,
  ],
});

app.get('/', (req, res) => {
  const jobs = {
    MyComponent: { name: req.query.name || 'Stranger' },
    Component2: { text: 'Hello World' },
  };

  renderer.render(jobs).then(html => res.send(html));
});

app.listen(8080, () => console.log('Now listening'));
```

## Plugin Lifecycle API

```typescript
function getViewData(viewName: string, data: any): any {}
```

Allows you to alter the data that a "view" will receive.

```typescript
type Job = { name: string, data: any };
type Jobs = { [string]: Job };
function prepareRequest(currentJobs: Jobs, originalJobs: Jobs): Jobs {}
```

A reducer type function that is called when preparing the request that will be sent to Hypernova.
This function receives the current running jobs Object and the original jobs Object.

```typescript
function shouldSendRequest(jobs: Jobs): boolean {}
```

An `every` type function. If one returns `false` then the request is canceled.

```typescript
function willSendRequest(jobs: Jobs): void {}
```

An event type function that is called prior to a request being sent.

```typescript
type Job = { name: string, data: any };
type Response = {
  [string]: {
    error: ?Error,
    html: string,
    job: Job,
  },
};
function afterResponse(currentResponse: any, originalResponse: Response): any {}
```

A reducer type function which receives the current response and the original response from the
Hypernova service.

```typescript
type Job = { name: string, data: any };
type Jobs = { [string]: Job };
function onSuccess(jobs: Jobs): void {}
```

An event type function that is called whenever a request was successful.

```typescript
type Job = { name: string, data: any };
type Jobs = { [string]: Job };
function onError(err: Error, jobs: Jobs): void {}
```

An event type function that is called whenever any error is encountered.
