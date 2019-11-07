# bookmark
https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render


# LINKS
  - [Main ref](https://reactjs.org/docs/react-api.html)
  - [React.Component API reference](https://reactjs.org/docs/react-component.html)
  - [Hooks tutorial](https://reactjs.org/docs/hooks-intro.html)
  - [hooks FAQ](https://reactjs.org/docs/hooks-faq.html)

## articles
  - [react.cloneelement vs children](https://stackoverflow.com/questions/37521798/when-should-i-be-using-react-cloneelement-vs-this-props-children/50441271#50441271)
  - [transforming elements in react](https://medium.com/javascript-inside/transforming-elements-in-react-8e411c0f1bba)
  -
#  COMPONENTS
```js
  // React.Component
  // React.PureComponent
```

## DYNAMIC
```js
  // React.lazy
  // React.Suspense
```

# REFS
```js
  // React.createRef
  // React.forwardRef
```

# Hooks
  - use cases
    - extract stateful logic from a component so it
      - can be tested independently
      - reused between components without changing component hierarchy
    - provide access to imperative escape hatches
    -

  - why not classes
    - dont minify well
    - make hot reloading flaky and unreliable

  - gotchas
    - no equivalents to getSnapshotBeforeUpdate or componentDidCatch
    - Any function inside a component, including event handlers and effects, “sees” the props and state from the render it was created in.

  - react redux
    - support v7.1.0
    - exposes useDispatch and useSelector
  - react router
    - supports hooks since v5.1
  - react-dom/test-utils
    - provides { act }
  - eslint plugin exists
    - enforces rules of hooks to avoid bugs
    - [facebook eslint plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation)

## hooks: class migration
  - useRef
    - you can think of refs as similar to instance variables in a class.
    - Unless you’re doing lazy initialization, avoid setting refs during rendering — this can lead to surprising behavior.
    - Instead, typically you want to modify refs in event handlers and effects.
    - If you intentionally want to read the latest state from some asynchronous callback, you could keep it in a ref, mutate it, and read from it.
    - gotchas
      - an object ref doesn’t notify us about changes to the current ref value
        - use a callback ref instead

  - callback ref
    - to measure the position or size of a DOM node, you can use a callback ref. React will call that callback whenever the ref gets attached to a different node.
    - Using a callback ref ensures that even if a child component displays the measured node later (e.g. in response to a click), we still get notified about it in the parent component and can update the measurements.

  - getDerivedStateFromProps
    - schedule an update while rendering
  - shouldComponentUpdate
    - wrap a function component with React.memo to shallowly compare its props:


## Hooks - types
  - best practices
    - declare functions needed by an effect inside of it
    - specify a list of dependencies as the last argument to
      - useEffect
      - useMemo
      - useCallback
      - useImperativeHandle
      - requirements
        - must include all values used inside the hook that participate in the react data flow
          - props
          - state
          - anything derived from them


  - notes
    - Both useState and useReducer Hooks bail out of updates if the next value is the same as the previous one. Mutating state in place and calling setState will not cause a re-render.
    - Hook calls can’t be placed inside loops.


  - effect hook
    - setting up subscriptings
    - fetching data
    - replaces lifecycle methods
      - componentDidMount
      - componentDidUpdate
      - componentWillUnmount
    - create an instance variable with useRef
    - If you need it, you can use a mutable ref to manually store a boolean value corresponding to whether you are on the first or a subsequent render, then check that flag in your effect. (If you find yourself doing this often, you could create a custom Hook for it.)
    - create functions inside the effect
      - if unable to
        - move function outside of the component
          - guaranteed not to reference any props/state
          - doesnt need to be on list of deps
        - if its a pure computation and safe to call while rendering
          - call it outside of the effect
          - make the effect depend on the return value
        - add the function to effect deps but wrap its definition into the useCallback hook
          - ensures it doesnt change on every render
    - The empty set of dependencies, [], means that the effect will only run once when the component mounts, and not on every re-render.


  - reducer
    - managing local state
    - If the state logic becomes complex, we recommend managing it with a reducer or a custom Hook.


  - useState
    - initialize state
    - for expensive computation, pass a function to useState
    - This is because when we update a state variable, we replace its value.
      - This is different from this.setState in a class, which merges the updated fields into the object.
      - we recommend to split state into multiple state variables based on which values tend to change together.
    - the functional update form of setState. It lets us specify how the state needs to change without referencing the current state:
      - is only called during the first render


  - useImperativeHandle
    - make a ref to a function component
    - expose a method to a parent component

  - useMemo
    - cache calculations between multiple renders by “remembering” the previous computation
    - Remember that the function passed to useMemo runs during rendering. Don’t do anything there that you wouldn’t normally do while rendering.
    -  lets you skip an expensive re-render of a child:



```js
  // create an instance variable
    function Timer() {
        const intervalRef = useRef();

        useEffect(() => {
          const id = setInterval(() => {
            // ...
          });
          intervalRef.current = id;
          return () => {
            clearInterval(intervalRef.current);
          };
        });

      // ...
    }
  // get prev props/state
      function Counter() {
        const [count, setCount] = useState(0);

        const prevCountRef = useRef();
        useEffect(() => {
          prevCountRef.current = count;
        });
        const prevCount = prevCountRef.current;

        return <h1>Now: {count}, before: {prevCount}</h1>;
      }
  // get previous props/state with custom hook
      function Counter() {
        const [count, setCount] = useState(0);
        const prevCount = usePrevious(count);
        return <h1>Now: {count}, before: {prevCount}</h1>;
      }

      function usePrevious(value) {
        const ref = useRef();
        useEffect(() => {
          ref.current = value;
        });
        return ref.current;
      }
  // force a rerender by incrementing a counter
  // avoid this if possible
      const [ignored, forceUpdate] = useReducer(x => x + 1, 0);

       function handleClick() {
         forceUpdate();
       }

  // measure a dom node
    function MeasureExample() {
      const [height, setHeight] = useState(0);

      const measuredRef = useCallback(node => {
        if (node !== null) {
          setHeight(node.getBoundingClientRect().height);
        }
      }, []);

      return (
        <>
          <h1 ref={measuredRef}>Hello, world</h1>
          <h2>The above header is {Math.round(height)}px tall</h2>
        </>
      );
    }

  // reusable hook to measure a dom node
      function MeasureExample() {
        const [rect, ref] = useClientRect();
        return (
          <>
            <h1 ref={ref}>Hello, world</h1>
            {rect !== null &&
              <h2>The above header is {Math.round(rect.height)}px tall</h2>
            }
          </>
        );
      }

      function useClientRect() {
        const [rect, setRect] = useState(null);
        const ref = useCallback(node => {
          if (node !== null) {
            setRect(node.getBoundingClientRect());
          }
        }, []);
        return [rect, ref];
      }
  // example specifying dependencies
      function ProductPage({ productId }) {
        const [product, setProduct] = useState(null);

        useEffect(() => {
          // By moving this function inside the effect, we can clearly see the values it uses.
          async function fetchProduct() {
            const response = await fetch('http://myapi/product' + productId);
            const json = await response.json();
            setProduct(json);
          }

          fetchProduct();
        }, [productId]); // ✅ Valid because our effect only uses productId
        // ...
      }

  // use a flag to specify when to fetch
      useEffect(() => {
        let ignore = false;
        async function fetchProduct() {
          const response = await fetch('http://myapi/product/' + productId);
          const json = await response.json();
          if (!ignore) setProduct(json);
        }

        fetchProduct();
        return () => { ignore = true };
      }, [productId]);

  // useEffect  + useCallback hook
    function ProductPage({ productId }) {
      // ✅ Wrap with useCallback to avoid change on every render
      const fetchProduct = useCallback(() => {
        // ... Does something with productId ...
      }, [productId]); // ✅ All useCallback dependencies are specified

      return <ProductDetails fetchProduct={fetchProduct} />;
    }

    function ProductDetails({ fetchProduct }) {
      useEffect(() => {
        fetchProduct();
      }, [fetchProduct]); // ✅ All useEffect dependencies are specified
      // ...
    }

  // use react.memo to shallow compare props
  // similar to shouldComponentUpdate
      const Button = React.memo((props) => {
        // your component
      });

  // useMemo
  // if a,b doesnt change, it returns previous known value
    const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

  // skip rerendering of child component
      function Parent({ a, b }) {
        // Only re-rendered if `a` changes:
        const child1 = useMemo(() => <Child1 a={a} />, [a]);
        // Only re-rendered if `b` changes:
        const child2 = useMemo(() => <Child2 b={b} />, [b]);
        return (
          <>
            {child1}
            {child2}
          </>
        )
      }

  // avvoid creating an object until its required
      function Image(props) {
        const ref = useRef(null);

        // ✅ IntersectionObserver is created lazily once
        function getObserver() {
          if (ref.current === null) {
            ref.current = new IntersectionObserver(onIntersect);
          }
          return ref.current;
        }

        // When you need it, call getObserver()
        // ...
      }
```