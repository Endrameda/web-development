---
tags:
  - react/patterns
status: growing
---
# React - Compound Components

> One widget split into a parent that owns the state and parts that consume it through context — the consumer composes JSX instead of configuring one big props object. 

Compound components are set of components which are dependent on each other through the shared state, and share logic together. Often we are seeing them as `select`, `menu item` , `dropdown components`, `card`. Compound Components pattern allows us to create component that all work together to perform a task.

## When to reach for it

Use it when a widget has several parts the consumer may want to arrange, restyle, or omit — Tabs, Select, Accordion, Wizard. The parent owns the state, the parts consume it implicitly through context, and this is exactly the API of Radix, Base UI and Headless UI.

The trade-off: composition wins when the *structure* varies between usages; a config prop (`<Menu items={[...]} />`) wins when the data is uniform and the structure never changes — a list of 200 identical rows does not need 200 JSX children. Rule of thumb: if consumers keep asking for one more prop to move or hide a part, switch to compound; if they only ever pass data, keep the config prop.

## Example
In internet we can find too many examples. We will implement the simple `Wizard`(Stepper) component which will demonstrate how we can use Compound Component in real life.

We start form the creating the context, because the main thing we are using Compound Components it's because of context and keeping all state inside it if possible.
Here we created a context with the basic setup for the `Wizard` component (You can setup how you want, depends on what feature you want to include to the component).
We also create in this file `useWizardContext` because it's best practices in ReactJS to keep hook and context together.

```TSX
interface WizardContextValue {
  registerStep: (name: string) => () => void;
  onComplete: () => void;
  goNext: () => void;
  goPrev: () => void;
  isFirstStep: boolean;
  isLastStep: boolean;
  activeStep: string | undefined;
}

const WizardContext = createContext<WizardContextValue | null>(null);

const useWizardContext = () => {
  const context = useContext(WizardContext);

  if (!context) {
    throw new Error("useWizardContext must be used within a WizardProvider");
  }
  return context;
};
```


Now time to create the `Root` Component. In the `Root` we will keep our main logic and functionality.

```TSX
interface RootProps extends PropsWithChildren {
  onComplete: () => void;
}

function Root(props: RootProps) {
  const { onComplete, children } = props;
  const [steps, setSteps] = useState<string[]>([]);
  const [currentStep, setCurrentStep] = useState<string | null>(null);

  const activeStep = currentStep ?? steps[0];
  const activeStepIndex =
    activeStep !== undefined ? steps.indexOf(activeStep) : -1;
  const isFirstStep = activeStepIndex === 0;
  const isLastStep = steps.length > 0 && activeStepIndex === steps.length - 1;

  const registerStep = useCallback((name: string) => {
    setSteps((prev) => (prev.includes(name) ? prev : [...prev, name]));
    return () => setSteps((prev) => prev.filter((s) => s !== name));
  }, []);

  const goNext = useCallback(() => {
    const next = steps[activeStepIndex + 1];
    if (next !== undefined) setCurrentStep(next);
  }, [activeStepIndex, steps]);

  const goPrev = useCallback(() => {
    const prev = steps[activeStepIndex - 1];
    if (prev !== undefined) setCurrentStep(prev);
  }, [activeStepIndex, steps]);

  const providerValues: WizardContextValue = useMemo(
    () => ({
      onComplete,
      registerStep,
      activeStep,
      isFirstStep,
      isLastStep,
      goNext,
      goPrev,
    }),
    [
      onComplete,
      registerStep,
      activeStep,
      isFirstStep,
      isLastStep,
      goNext,
      goPrev,
    ],
  );

  return (
    <WizardContext.Provider value={providerValues}>
      {children}
    </WizardContext.Provider>
  );
}

```

As you can see here we are wrapping the `Root`'s content with `WizardContext.Provider` and passing there values for the `Provider`

And now lets to move to another components which we can possibly need.
It will be the `Step` as you can see, we are using here `useWizardContext` from our `context` file. And from context we are taking our state, and functions
```TSX
interface StepProps extends PropsWithChildren {
  name: string;
}

function Step(props: StepProps) {
  const { name, children } = props;
  const { registerStep, activeStep } = useWizardContext();

  useEffect(() => registerStep(name), [registerStep, name]);

  if (activeStep !== name) return null;

  return <div>{children}</div>;
}
```

Now the Buttons, Prev, Next they are almost the same.
```TSX
function Prev(props: PropsWithChildren) {
  const { children } = props;
  const { goPrev, isFirstStep } = useWizardContext();

  return (
    <button onClick={goPrev} disabled={isFirstStep}>
      {children}
    </button>
  );
}
```

`Next` is the one place they differ: on the last step it becomes Finish and fires `onComplete` — from the click handler, not from an effect (see Gotchas).

```TSX
function Next(props: PropsWithChildren) {
  const { children } = props;
  const { goNext, isLastStep, onComplete } = useWizardContext();

  return (
    <button onClick={isLastStep ? onComplete : goNext}>
      {isLastStep ? "Finish" : children}
    </button>
  );
}
```

And finally the usage. 
BTW have a few ways to Map your compound components, here I just used the `const = {...}` it's not too good for the tree shaking which our bundler does while compiling. But anyway, it's just an example.

```TS
import Next from "./Next";
import Prev from "./Prev";
import Root from "./Root";
import Step from "./Step";

export const Wizard = {
  Root,
  Step,
  Next,
  Prev,
};

```

The Usage

```TSX
function App() {
  const handleOnComplete = () => {
    console.log("On Complete is Fired");
  };

  return (
    <Wizard.Root onComplete={handleOnComplete}>
      <Wizard.Step name="step-1">Step 1</Wizard.Step>
      <Wizard.Step name="step-2">Step 2</Wizard.Step>
      <Wizard.Step name="step-3">Step 3</Wizard.Step>
      <Wizard.Prev>Prev</Wizard.Prev>
      <Wizard.Next>Next</Wizard.Next>
    </Wizard.Root>
  );
}
```

It's just an example and what I understand while learning before. 
Additionally below I have sources, where you can check and go more deep to learn Compound Component.

And will be more better if you will fork some UI library, [BaseUI](https://github.com/mui/base-ui) and go to their code base. you can see more realistic examples of the Compound Components. 
## Gotchas

All of these are bugs I actually hit while building the Wizard:

- **`setState(!value)` is a stale closure.** Always use the functional updater: `setValue(prev => !prev)`. Two rapid updates in one tick otherwise collapse into one.
- **Unguarded `useContext` destructure.** A part rendered outside its Root gets `undefined` and dies with "cannot destructure". The guard hook that throws a named error (`useWizardContext` above) is the standard fix — Radix generates exactly this via its own `createContext` factory.
- **Side effects belong to events, not render or effects.** I fired `onComplete()` from the render body first, then from a `useEffect` watching `isLastStep` — both wrong. Completing is a click. Watching state in an effect to fire a callback is the classic "You Might Not Need an Effect" mistake.
- **Empty collections lie.** With no steps registered yet, `indexOf` gives `-1` and `steps.length - 1` gives `-1`, so an empty wizard reported being on its last step — and auto-completed on page load. Guard derived booleans with `steps.length > 0`.
- **The part must stay mounted; only its content is conditional.** If `Step` unmounted while inactive, its registration cleanup would run and the wizard would forget the step exists. `Step` always renders and returns `null` for its content instead.
- **StrictMode double-invocation is a bug detector, not an annoyance.** It caught both the double registration (fixed by cleanup + dedupe) and the auto-complete on load. Do not turn it off.
- **`noUncheckedIndexedAccess`.** `steps[0]` on an empty array is `undefined`, but TypeScript types it `string` by default. With this flag on, my `activeStep: string` type would have been a compile error instead of a runtime surprise.
- **Context value must be memoized.** An inline `value={{ ... }}` is a new object every render, so every consumer re-renders every time. `useMemo` it; at scale, split fast-changing state and static config into separate contexts.

Full implementation with git history: `web-development-vault-examples/react-compound-components` (local for now — replace with the GitHub link once pushed).

## Sources

- [patterns.dev — Compound Pattern](https://www.patterns.dev/react/compound-pattern/)
- [Kent C. Dodds — Compound Components with React Hooks](https://kentcdodds.com/blog/compound-components-with-react-hooks)
- [Vercel Academy — Compound Components and Advanced Composition](https://vercel.com/academy/shadcn-ui/compound-components-and-advanced-composition) — best source to go deeper
- [Base UI source](https://github.com/mui/base-ui) — realistic implementations; note `export * as Select` in `select/index.ts` is the modern answer to `X.Part` naming (module namespace, not runtime statics)
