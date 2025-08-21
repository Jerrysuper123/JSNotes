# JSNotes

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
