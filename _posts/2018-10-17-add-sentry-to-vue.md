---
layout: post
title:  "Add Sentry to a Vue project"
subtitle: "It's easier than you think"
author: heisenberg.yoon
background: '/img/posts/05.jpg'
published: false
---

<p align="center">
  <a href="https://sentry.io" target="_blank" align="center">
    <img src="https://sentry-brand.storage.googleapis.com/sentry-logo-black.png" width="280">
  </a>
  <br />
</p>

# How to add sentry to VueJS project

This is a document to describe how to set up `@sentry-browser` into a VueJS project

## Contents

- [Why Sentry?](#why-sentry?)
- [Preview](#preview)
  - [Error summary](#error-summary)
  - [Breadcrumbs](#breadcrumbs)
  - [User info](#user-info)
  - [Additional data](#additional-data)
- [Getting started](#getting-started)
- [Installation and Usage](#installation-and-usage)
- [Examples](#examples)
  - [Configuring user information](#configuring-user-information)
  - [Reporting Axios error to Sentry](#reporting-axios-error-to-sentry)

## Why Sentry?

Sentry can capture custom errors and events defined by developers and send email notifications to the stakeholders. You don't even have to ask when, how an error occured in which environment.

## Preview

### Error summary

![image.png](/.attachments/image-fc3df7dd-352c-4f8f-a619-966509c1ef0c.png)

### Breadcrumbs

![image.png](/.attachments/image-218cb540-bd1e-4ff4-bb6d-a52a342470bc.png)

### User info

![image.png](/.attachments/image-d249320e-a44d-4b2e-bd8e-5b1a10dd5872.png)

### Additional data

![image.png](/.attachments/image-685b664b-5d89-4aec-8d51-38e77e1faa7b.png)

## Getting started

1. Sign in on [sentry.io](http://sentry.io).

![image.png](/.attachments/image-158511c3-9497-4253-920a-03dfdcda7347.png)

```text
```

2. Click on Add new... on top right and add a Vue project.
![image.png](/.attachments/image-9b9a6765-62a5-42ef-99d8-7bdddb82e0e8.png)
![image.png](/.attachments/image-26b4ec69-d50f-4ad9-823c-fdaa11531977.png)

3. Get a DSN 
   - After you completed setting up a project in Sentry, you’ll be given a value which we call a **DSN**, or *Data Source Name*. It looks a lot like a standard URL, but it’s actually just a representation of the configuration required by the Sentry SDKs. It consists of a few pieces, including the protocol, public key, the server address, and the project identifier.
Sign in on [sentry.io](http://sentry.io) and open [DSN doc link](https://docs.sentry.io/quickstart/?platform=browser). Then can find your DSN. Alternatively, the DSN can be found in Sentry by navigating to `[Project Name] -> Project Settings -> Client Keys (DSN)`. Its template resembles the following:

      `'{PROTOCOL}://{PUBLIC_KEY}@{HOST}/{PROJECT_ID}'`

## Installation and Usage

To install a SDK, simply add the high-level package, for example:

```sh
npm install --save @sentry/browser
yarn add @sentry/browser
```

Setup and usage of these SDKs follows the same principle.

1. In `@/plugins/sentry.js`:

```javascript
import Vue from 'vue'
import * as Sentry from '@sentry/browser'
import { name, version } from '@/../package.json' // To log project name and version

Sentry.init({
  release: `${name}@${version}`,
  dsn: '__DSN__',
  // This is optional, but too many consoles (e.g. Vuex debugger) may freeze the browser
  beforeBreadcrumb (breadcrumb) {
    return breadcrumb.category === 'console' ? null : breadcrumb
  },
  // Use default Sentry integration for vue
  integrations: [new Sentry.Integrations.Vue({
    Vue
  })]
})

// Optionally, you can define sentry globally
Object.defineProperty(Vue.prototype, '$sentry', { value: Sentry })

export default Sentry
```

2. In `@/main.js`

```javascript
import Vue from 'vue'
(...)
import './plugins/sentry'
import './plugins/axios'
(...)
```

## Examples

### Configuring user information

It's always to know about the user who happened to report an error. Browser info and IP address will be recorded automatically. However it is also easy to set up user information gloablly.

- In your login callback,

```javascript
import Sentry from '@/plugins/sentry'

afterLogin (payload) {
  Sentry.configureScope((scope) => {
    scope.setUser({
      id: payload.user_id,
      email: payload.email
    })
  })
}
```

### Reporting Axios error to Sentry

XHR errors are captured by default, but you can customize it to see what really happened.

- In `@/plugins/axios.js`:

```javascript
import Vue from 'vue'
import Axios from 'axios'
import Sentry from './sentry'
(...)

Axios.interceptors.response.use(
  response => response,
  (error) => {
    // Set up an error object to be sent to Sentry
    const { response } = error
    const { config } = response
    const errorObj = {
      endpoint: config.url,
      payload: JSON.parse(config.data),
      status: response.status,
      response: response.data
    }
    // Send an error to Sentry with the error object
    // Sentry.captureMessage('Axios Error!')
    Sentry.captureException(errorObj)
    return Promise.reject(error)
  }
)
```