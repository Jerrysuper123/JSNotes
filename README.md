# JSNotes

Got it 👍 — so now you’ve got **two layers of try/catch**:

1. **Outer layer** → catches network errors from `fetch`.
2. **Inner layer (inside `downloadInstaller`)** → catches errors that happen while handling the response (e.g., parsing, saving, UI updates).

That’s actually a good pattern 👌. Let me show you how to wire it cleanly.

---

### Example with async/await

```js
async function startDownload() {
  try {
    // Outer try/catch: catch network errors (e.g. blocked URL, DNS failure, timeout)
    const response = await fetch(signedRequest);

    // Still check HTTP status, since fetch does not throw for 4xx/5xx
    if (!response.ok) {
      throw new Error(`HTTP ${response.status} - ${response.statusText}`);
    }

    // Call your installer logic
    await downloadInstaller(
      response,
      invokeAction,
      installerFileName,
      app,
      fleetId,
      getAuthClient,
      installKeyOCID,
    );
  } catch (err) {
    console.error("❌ Outer error (network or fetch):", err);
    // maybe show UI error here
  }
}

async function downloadInstaller(
  response,
  invokeAction,
  installerFileName,
  app,
  fleetId,
  getAuthClient,
  installKeyOCID
) {
  try {
    // Example: actually consume the response
    const blob = await response.blob();

    // Do your custom logic here
    console.log("✅ Downloaded", installerFileName, "size:", blob.size);

    // ... more code for saving, executing, etc.
  } catch (err) {
    console.error("❌ Inner error (handling response):", err);
    // handle UI cleanup, retry logic, etc.
  }
}
```

---

### Flow explained

* If the **network request is blocked** (like your Chrome blocked URL case) → outer `catch` runs.
* If the network succeeds but something fails inside `downloadInstaller` (bad response format, disk write issue, custom logic crash) → inner `catch` runs.

---

### Alternative with `.then().catch()`

If you don’t want to use `await` in `startDownload`, you can write:

```js
fetch(signedRequest)
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return downloadInstaller(
      response,
      invokeAction,
      installerFileName,
      app,
      fleetId,
      getAuthClient,
      key,
    );
  })
  .catch((err) => {
    console.error("❌ Outer error:", err);
  });
```

---

✅ So the rule of thumb:

* **Outer `try/catch` / `.catch()`** → network/fetch-level errors.
* **Inner `try/catch` inside `downloadInstaller`** → processing-level errors.

---


JS and React are just weird sometimes, and it is hard to remember some of its underlying concepts. So this article records those for better memory reinforcement.

# how to capture error at different level
You’ll only ever see err.name === "AbortError" if you actually use an AbortController with fetch.

That’s because:

Normal HTTP errors (4xx, 5xx) → fetch does not throw. You must check response.ok.

Network/firewall errors (DNS fail, connection refused, CORS issue, etc.) → fetch rejects with a TypeError, not AbortError.

Timeouts / cancellations (when you call controller.abort()) → fetch rejects with an error where err.name === "AbortError".

```
async function loadData(timeoutMs = 10000) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const response = await fetch("/api/data", { signal: controller.signal });

    if (!response.ok) {
      let errorBody: any = {};
      try {
        errorBody = await response.json();
      } catch {
        errorBody = { message: response.statusText };
      }

      throw {
        type: "HttpError",
        status: response.status,
        code: errorBody.code || "UNKNOWN_ERROR",
        message: errorBody.message || "Request failed",
      };
    }

    return await response.json();
  } catch (err: any) {
    if (err.name === "AbortError") {
      // Timeout or manually aborted
      throw {
        type: "TimeoutError",
        message: `Request timed out after ${timeoutMs}ms`,
      };
    }

    // Network / firewall error (fetch couldn’t connect at all)
    if (err instanceof TypeError) {
      throw {
        type: "NetworkError",
        message: "Network error or firewall blocked request",
      };
    }

    // Re-throw unknown error
    throw err;
  } finally {
    clearTimeout(timeout);
  }
}

```

# How props are used in React?

We often create a React component below

```
Const ReactComponent = (propsAnything: {id: number; name: string; size: string})=>{

Const name = propsAnything.name;
}
```


## how we use spread operator instead of typing every key value for the props
```
const row = {
  id: 1,
  name: 'file',
  size: '10MB'
};


<ReactComponent {...row} />
// {...row} helps to spread to convert into below
// question: what is the use {}, why do we need it?
< ReactComponent id={1} name="file" size="10MB" />.
```
The above case is just weird because it means `id={1} name="file" size="10MB"` would be wrapped as an props object to be injected into the React component `{
  id: 1,
  name: 'file',
  size: '10MB'
};`

## what is the use of `{...row}`?
{...row} dynamically passes all the properties of the row object as individual props to ReactComponent.
It simplifies passing multiple props, especially when they come from an object, and reduces redundancy.
So ... means to spread the object into individual key value?
Then {} means to wrap the key value into an object?

## destructuring an object
Destructing object is not new in JS.
```
const obj = { a: 1, b: { c: 2 } };
const { a } = obj; // a is 1

//so similarly
Const {id, name, size} = propsAnything;
```

Based on above, we could destructure at the parameter `propsAnything` itself right away.
- **Cleaner code**: You no longer need to repeatedly reference `propsAnything.name`, `propsAnything.id`, etc.
- **Clarity**: When you destructure in the parameter list, it's immediately clear which props are being used in the component.
- **Less verbose**: It simplifies the code when you're using multiple props.
```
Const ReactComponent = (propsAnything: {id: number; name: string; size: string})=>{}
Const ReactComponent = ({id, name, size}: {id: number; name: string; size: string})=>{}
```
