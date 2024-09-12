# JSNotes


```
Const ReactComponent = (propsAnything: {id: number; name: string; size: string})=>{

Const name = propsAnything.name;
}
```


### Example:
```jsx
const row = {
  id: 1,
  name: 'file',
  size: '10MB'
};

<ReactComponent {...row} />
```
`< ReactComponent id={1} name="file" size="10MB" />`.


# destructuring an object
Const ReactComponent = ({id, name, size}: {id: number; name: string; size: string})=>{

# you can use name direct from props destructuring
}

- **Cleaner code**: You no longer need to repeatedly reference `propsAnything.name`, `propsAnything.id`, etc.
- **Clarity**: When you destructure in the parameter list, it's immediately clear which props are being used in the component.
- **Less verbose**: It simplifies the code when you're using multiple props.
