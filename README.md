# Network Handling and Monitoring in Playwright (JavaScript)

Here we are handling concepts like request interception, mocking API responses, modifying requests, response stubbing, and tracking network activity, all using JavaScript. We'll explain each concept with code snippets and walk through a complete example combining these features. By the end, you will know how to monitor and manipulate network requests in your Playwright tests.

## What is Network Interception in Playwright?

Network interception means catching web requests that a page makes before they reach the server or as they come back, and deciding how to handle them. With Playwright, you can monitor every network request and response, and even modify or mock them.

This is useful for:

* Simulating slow or failing network calls
* Testing app behavior with specific API data
* Speeding up tests by blocking unnecessary resources (images, ads)

## Intercepting Requests with `page.route()`

Using `page.route()`, you can intercept network requests and decide how to handle them:

* Continue the request
* Fulfill the request with mock data
* Abort the request entirely

```js
await page.route('**/*.{png,jpg,jpeg}', route => route.abort());
await page.goto('https://example.com');
```

This intercepts all image requests and blocks them.

## Mocking API Responses with `route.fulfill()`

Use `route.fulfill()` to mock API responses.

```js
await page.route('*/**/api/v1/fruits', async route => {
  const fakeData = [ { name: 'Strawberry', id: 21 } ];
  await route.fulfill({ 
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify(fakeData)
  });
});

await page.goto('https://demo.playwright.dev/api-mocking');
await expect(page.getByText('Strawberry')).toBeVisible();
```

## Modifying Requests with `route.continue()`

Use `route.continue()` to modify requests before they are sent to the server.

```js
await page.route('**/*', async route => {
  const request = route.request();
  const headers = { ...request.headers() };
  delete headers['X-Secret'];
  await route.continue({ headers });
});

await page.goto('https://example.com');
```

## Stubbing and Modifying Responses

You can fetch a real response with `route.fetch()` and modify it before continuing.

```js
await page.route('*/**/api/v1/fruits', async route => {
  const response = await route.fetch();
  let fruits = await response.json();
  fruits.push({ name: 'Loquat', id: 100 });
  await route.fulfill({
    response,
    json: fruits
  });
});

await page.goto('https://demo.playwright.dev/api-mocking');
await expect(page.getByText('Loquat')).toBeVisible();
```

## Tracking Network Activity

Playwright emits network events you can listen to:

* `request`
* `response`
* `requestfailed`
* `requestfinished`

```js
page.on('request', request => {
  console.log(`>> [Request] ${request.method()} ${request.url()}`);
});

page.on('response', response => {
  console.log(`<< [Response] ${response.status()} ${response.url()}`);
});

await page.goto('https://example.com');
```

## Waiting for a Specific Network Call

```js
const [response] = await Promise.all([
  page.waitForResponse('**/api/fetch_data'),
  page.click('text=Update')
]);
console.log("Update API response status:", response.status());
```

## End-to-End Example

```js
import { test, expect } from '@playwright/test';

test('Example: network interception and monitoring', async ({ page }) => {
  page.on('request', req => {
    console.log(`Request: ${req.method()} ${req.url()}`);
  });
  page.on('response', res => {
    console.log(`Response: ${res.status()} ${res.url()}`);
  });

  await page.route('**/*.{png,jpg,jpeg}', route => route.abort());

  await page.route('**/api/v1/fruits', route => {
    const testFruits = [{ name: 'Pineapple', id: 99 }];
    route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify(testFruits)
    });
  });

  await page.goto('https://demo.playwright.dev/api-mocking');
  await page.waitForResponse(resp => resp.url().includes('/api/v1/fruits'));
  await expect(page.getByText('Pineapple')).toBeVisible();
});
```

## Additional Learning Resources

* [Playwright Network Docs](https://playwright.dev/docs/network)
* [API Mocking Guide](https://playwright.dev/docs/network#mocking-apis)
* [Checkly Blog on Speeding Up Tests](https://www.checklyhq.com/blog/playwright-request-interception/)
* [Tim Deschryver’s Guide](https://timdeschryver.dev/blog/intercepting-http-requests-with-playwright)

This guide gives you the essential tools and examples to begin mastering network handling in Playwright using JavaScript.

