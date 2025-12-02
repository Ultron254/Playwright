# Network Handling and Monitoring in Playwright (JavaScript)

I shall be taking you through the concepts like request interception, mocking API responses, modifying requests, response stubbing, and tracking network activity, all using JavaScript. We'll explain each concept with code snippets and walk through a complete example combining these features. By the end, you will know how to monitor and manipulate network requests in your Playwright tests.

## What is Network Interception in Playwright?

Network interception means catching web requests that a page makes before they reach the server or as they come back, and deciding how to handle them. With Playwright, you can monitor every network request and response, and even modify or mock them.

This is useful for:

* Simulating slow or failing network calls
* Testing app behavior with specific API data
* Speeding up tests by blocking unnecessary resources (images, ads)

## Intercepting Requests with `page.route()`

Use this when you want to intercept specific requests for debugging, blocking, or rerouting purposes. This is commonly used to block third-party content or image assets that slow down test runs.

```js
await page.route('**/*.{png,jpg,jpeg}', route => route.abort());
await page.goto('https://example.com');
```

This intercepts all image requests and blocks them.

## Mocking API Responses with `route.fulfill()`

Mocking allows you to simulate API endpoints that might be slow, unstable, or still under development. You can test how your frontend behaves under controlled conditions.

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

Modify outgoing requests when you want to test how your app responds to different authentication tokens, headers, or payload data. This is useful for security testing, A/B testing, and conditional logic validation.

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

Use this when the actual API works but you want to inject custom data into the response. For instance, adding a test item to a product list or simulating a data structure change.

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

Tracking lets you debug requests that are being made and see exactly what your frontend is sending and receiving. This is essential for test validation, performance analysis, and failure diagnosis.

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

Waiting ensures your test only proceeds once a particular call has been made and completed. This is important when actions like button clicks trigger asynchronous API requests.

```js
const [response] = await Promise.all([
  page.waitForResponse('**/api/fetch_data'),
  page.click('text=Update')
]);
console.log("Update API response status:", response.status());
```

## End-to-End Example

This test demonstrates intercepting a request, mocking its response, and verifying UI behavior — all while monitoring the network.

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
