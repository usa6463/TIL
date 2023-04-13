# ReactJS Basic

## keyword
- component
- props
- state
  - state is private to a component that defines it
- shared state
  - lifting state up
- event handler
- arrow function: () => {}
- convention
  - onSomething: event
  - handleSomething: event handler
- re-render
  - all child components re-render automatically when the state of a parent component changes
  - [memo](https://react.dev/reference/react/memo)
- key property
  - key is a special and reserved property in React. When an element is created, React extracts the key property and stores the key directly on the returned element.