# 🎯 React Senior Interview — Hỏi & Đáp

> **Format:** Interviewer hỏi → Senior trả lời ngắn gọn, đi thẳng vào bản chất.

---

## Phần 1: React Internals

---

### Q1. Virtual DOM hoạt động như thế nào? Tại sao React không thao tác trực tiếp Real DOM?

**Senior:** React tạo một bản sao in-memory của DOM gọi là Virtual DOM. Khi state/props thay đổi, React tạo một Virtual DOM mới, so sánh (diff) với bản cũ bằng **Reconciliation Algorithm**, rồi chỉ cập nhật những node thực sự thay đổi lên Real DOM.

Lý do không thao tác trực tiếp: Real DOM operations rất tốn kém — mỗi lần thay đổi DOM, browser phải recalculate style, layout, repaint. Batching các thay đổi qua Virtual DOM giảm đáng kể số lần browser phải làm việc.

---

### Q2. React Fiber là gì? Nó giải quyết vấn đề gì so với Stack Reconciler cũ?

**Senior:** Fiber là engine reconciliation mới từ React 16. Stack Reconciler cũ xử lý render **đồng bộ** — một khi bắt đầu render tree thì phải chạy hết, block main thread.

Fiber chia render work thành các **unit of work** nhỏ, có thể:
- **Pause** giữa chừng để xử lý user input
- **Prioritize** urgent updates (click, type) trước non-urgent updates (data fetch)
- **Abort** work không còn cần thiết

Đây là nền tảng cho Concurrent Features trong React 18.

---

### Q3. Giải thích cơ chế Diffing Algorithm của React.

**Senior:** React dùng 2 heuristic chính:
1. **Khác type** → tear down cây cũ, build cây mới hoàn toàn.
2. **Cùng type** → so sánh attributes/props, chỉ update phần thay đổi. Với children list → dùng **`key`** để match elements.

Complexity: O(n) thay vì O(n³) của generic tree diff — trade-off accuracy lấy performance.

---

### Q4. Khi nào React quyết định re-render một component? Liệt kê đầy đủ.

**Senior:** 5 trường hợp:
1. **`setState()` / `useState` setter** được gọi
2. **Props thay đổi** (parent truyền props mới)
3. **Parent re-render** → children re-render (dù props không đổi, trừ khi dùng `React.memo`)
4. **Context value thay đổi** → tất cả consumers re-render
5. **`forceUpdate()`** (class component)

> Lưu ý: Re-render ≠ DOM update. React có thể re-render nhưng diff ra không có thay đổi → không touch DOM.

---

## Phần 2: Hooks Nâng Cao

---

### Q5. `useEffect` vs `useLayoutEffect` — khi nào dùng cái nào?

**Senior:**

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| **Thời điểm chạy** | Sau paint (async) | Sau DOM mutation, trước paint (sync) |
| **Use case** | Data fetching, subscriptions, logging | DOM measurement, scroll position, tooltip positioning |
| **Ảnh hưởng UX** | Không block paint | Block paint → dùng sai sẽ gây jank |

**Rule of thumb:** Luôn dùng `useEffect` trước. Chỉ chuyển sang `useLayoutEffect` khi thấy **visual flicker**.

---

### Q6. Giải thích "stale closure" trong hooks và cách fix.

**Senior:** Khi closure capture giá trị state tại thời điểm render, nhưng callback chạy sau khi state đã thay đổi:

```javascript
// ❌ Bug: count luôn = 0
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // closure capture count = 0
  }, 1000);
  return () => clearInterval(id);
}, []);

// ✅ Fix 1: Functional update
setCount(prev => prev + 1);

// ✅ Fix 2: useRef để track giá trị mới nhất
const countRef = useRef(count);
countRef.current = count;
setInterval(() => setCount(countRef.current + 1), 1000);

// ✅ Fix 3: Thêm count vào dependency array (nhưng cần clear interval đúng)
```

---

### Q7. `useMemo` và `useCallback` — khi nào KHÔNG nên dùng?

**Senior:** **Không nên dùng khi:**
- Computation **rẻ** (filter array nhỏ, string format) — overhead của memoization > cost của re-compute.
- Component **không được wrap** bởi `React.memo` — memoize callback vô nghĩa vì child vẫn re-render.
- Value/function **không được truyền xuống child** hoặc không nằm trong dependency array khác.

**Nên dùng khi:**
- Expensive computation (sort/filter large array, complex calculation)
- Truyền callback/object cho child đã `React.memo`
- Value nằm trong dependency array của `useEffect` khác

> **Premature optimization is the root of all evil.** Profile trước, optimize sau.

---

### Q8. Custom hook có gì khác so với một function bình thường?

**Senior:** Về bản chất kỹ thuật, custom hook **là** một function bình thường — nhưng tuân theo **Rules of Hooks**:
- Tên bắt đầu bằng `use` (convention để React lint check)
- Có thể gọi các hooks khác bên trong (useState, useEffect, ...)
- Mỗi component gọi hook sẽ có **instance state riêng** — không share state giữa các component

```javascript
function useDebounce(value, delay) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}
```

---

### Q9. `useRef` ngoài việc truy cập DOM thì dùng để làm gì?

**Senior:** `useRef` là một **mutable container** tồn tại xuyên suốt lifecycle, **không trigger re-render** khi thay đổi:

- **Lưu previous value:** `prevRef.current = value` trong useEffect
- **Lưu timer/interval ID:** tránh re-create khi re-render
- **Flag để track mount status:** `isMounted.current = true`
- **Lưu instance variable** như class component (any mutable value)

```javascript
// Track previous value
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; });
  return ref.current;
}
```

---

### Q10. `useReducer` vs `useState` — ranh giới nào để chọn?

**Senior:**

| Dùng `useState` | Dùng `useReducer` |
|---|---|
| State đơn giản (boolean, string, number) | State phức tạp (object nhiều field liên quan) |
| Ít state transitions | Nhiều action types khác nhau |
| Logic update đơn giản | Next state phụ thuộc vào previous state phức tạp |
| Không cần test logic riêng | Muốn test state logic **tách biệt** khỏi component |

```javascript
// Khi state transitions phức tạp → useReducer rõ ràng hơn
function reducer(state, action) {
  switch (action.type) {
    case 'FETCH_START': return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS': return { ...state, loading: false, data: action.payload };
    case 'FETCH_ERROR': return { ...state, loading: false, error: action.error };
  }
}
```

---

## Phần 3: Performance Optimization

---

### Q11. Làm sao bạn debug và fix performance issue trong React app?

**Senior:** Quy trình của tôi:

1. **React DevTools Profiler** → xác định component nào re-render nhiều nhất và mất bao lâu
2. **"Highlight updates when components render"** → thấy visual cái nào đang re-render
3. **Chrome Performance tab** → tìm long tasks, layout thrashing
4. **`why-did-you-render`** library → log nguyên nhân re-render chi tiết

Sau khi xác định bottleneck:
- Unnecessary re-render → `React.memo`, `useMemo`, `useCallback`
- Large list → **Virtualization** (react-window, react-virtuoso)
- Heavy computation → **Web Worker** hoặc `useDeferredValue`
- Large bundle → **Code splitting** (`React.lazy` + `Suspense`)

---

### Q12. `React.memo` hoạt động thế nào? Khi nào nó KHÔNG hoạt động?

**Senior:** `React.memo` wrap component với **shallow comparison** trên props. Nếu props không đổi → skip re-render.

**Không hoạt động khi:**
```javascript
// ❌ Object/array mới mỗi render → luôn khác reference
<Child style={{ color: 'red' }} />    // ❌ object mới
<Child items={data.filter(x => x)} /> // ❌ array mới
<Child onClick={() => doSth()} />     // ❌ function mới

// ✅ Fix
const style = useMemo(() => ({ color: 'red' }), []);
const filtered = useMemo(() => data.filter(x => x), [data]);
const onClick = useCallback(() => doSth(), []);
<Child style={style} items={filtered} onClick={onClick} />
```

Cũng không hoạt động khi component dùng **Context** — context value thay đổi thì `React.memo` bị bypass.

---

### Q13. Code splitting và lazy loading trong React — cách implement?

**Senior:**
```javascript
// Route-level splitting (phổ biến nhất)
const Dashboard = React.lazy(() => import('./Dashboard'));
const Settings = React.lazy(() => import('./Settings'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// Component-level splitting (cho heavy components)
const HeavyChart = React.lazy(() => import('./HeavyChart'));

// Named export splitting
const MyComponent = React.lazy(() =>
  import('./module').then(mod => ({ default: mod.MyComponent }))
);
```

Kết hợp với **Webpack magic comments** để prefetch:
```javascript
const About = React.lazy(() =>
  import(/* webpackPrefetch: true */ './About')
);
```

---

### Q14. Virtualization/Windowing là gì? Khi nào cần dùng?

**Senior:** Chỉ render các items **visible trong viewport**, thay vì render toàn bộ list. Với list 10,000 items, chỉ render ~20-30 items đang nhìn thấy.

**Dùng khi:** List > 100-200 items bắt đầu gây lag.

```javascript
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  return (
    <FixedSizeList
      height={600}
      width="100%"
      itemCount={items.length}
      itemSize={50}
    >
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>
      )}
    </FixedSizeList>
  );
}
```

Libraries: `react-window` (lightweight), `react-virtuoso` (feature-rich), `@tanstack/react-virtual` (headless).

---

## Phần 4: State Management

---

### Q15. Context API có vấn đề gì về performance? Cách giải quyết?

**Senior:** Khi context value thay đổi, **TẤT CẢ consumers re-render** — dù chúng chỉ dùng một phần nhỏ của value.

```javascript
// ❌ Mỗi khi theme HOẶC user thay đổi → cả 2 consumers re-render
const AppContext = createContext({ theme: 'dark', user: null });

// ✅ Fix 1: Tách context
const ThemeContext = createContext('dark');
const UserContext = createContext(null);

// ✅ Fix 2: Memoize context value
const value = useMemo(() => ({ theme, user }), [theme, user]);
<AppContext.Provider value={value}>

// ✅ Fix 3: Dùng selector pattern (Zustand, Jotai có built-in)
const theme = useStore(state => state.theme); // chỉ re-render khi theme đổi
```

---

### Q16. So sánh Redux Toolkit, Zustand, và Jotai — khi nào chọn cái nào?

**Senior:**

| | Redux Toolkit | Zustand | Jotai |
|---|---|---|---|
| **Kiến trúc** | Flux (single store, reducers, actions) | Single store, hooks-based | Atomic (bottom-up) |
| **Boilerplate** | Trung bình (RTK giảm nhiều) | Rất ít | Rất ít |
| **DevTools** | Xuất sắc | Tốt (middleware) | Tốt |
| **Bundle size** | ~11KB | ~1KB | ~2KB |
| **Khi nào dùng** | App lớn, team lớn, cần strict patterns | App vừa, cần đơn giản | State phân tán, derived state phức tạp |

**Rule of thumb:**
- Team lớn, cần convention → **Redux Toolkit**
- Cần nhanh gọn, ít ceremony → **Zustand**
- State atomic, nhiều derived values → **Jotai**
- Server state → **TanStack Query** (không phải Redux!)

---

### Q17. Server State vs Client State — tại sao cần phân biệt?

**Senior:** Đây là 2 loại state hoàn toàn khác bản chất:

| | Client State | Server State |
|---|---|---|
| **Ownership** | App sở hữu | Server sở hữu, app chỉ cache |
| **Ví dụ** | Theme, UI toggle, form input | User list, products, orders |
| **Tính chất** | Synchronous, predictable | Async, có thể stale, cần refetch |

Dùng **Redux/Zustand** cho client state. Dùng **TanStack Query / SWR** cho server state vì chúng giải quyết:
- Caching & deduplication
- Background refetching
- Stale-while-revalidate
- Pagination & infinite scroll
- Optimistic updates

```javascript
// TanStack Query — clean hơn rất nhiều so với useEffect + useState
const { data, isLoading, error } = useQuery({
  queryKey: ['users'],
  queryFn: () => fetch('/api/users').then(r => r.json()),
  staleTime: 5 * 60 * 1000, // 5 phút
});
```

---

## Phần 5: React 18+ & Concurrent Features

---

### Q18. Automatic Batching trong React 18 khác gì so với React 17?

**Senior:**

```javascript
// React 17: chỉ batch trong event handlers
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // ✅ 1 re-render (batched)
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // ❌ 2 re-renders (React 17 KHÔNG batch trong setTimeout)
}, 1000);

// React 18: batch EVERYWHERE — event handlers, setTimeout, promises, native events
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // ✅ 1 re-render (React 18 tự batch)
}, 1000);

// Muốn opt-out batching → flushSync
import { flushSync } from 'react-dom';
flushSync(() => setCount(c => c + 1)); // re-render ngay
flushSync(() => setFlag(f => !f));      // re-render ngay
```

---

### Q19. `useTransition` và `useDeferredValue` — giải quyết vấn đề gì?

**Senior:** Cả hai đều đánh dấu updates là **non-urgent**, cho phép React ưu tiên urgent updates (typing, clicking) trước.

```javascript
// useTransition — wrap state update
function SearchPage() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    setQuery(e.target.value);        // urgent: update input ngay
    startTransition(() => {
      setFilteredList(filter(data, e.target.value)); // non-urgent: có thể delay
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending ? <Spinner /> : <ResultList items={filteredList} />}
    </>
  );
}

// useDeferredValue — defer một value
function SearchResults({ query }) {
  const deferredQuery = useDeferredValue(query);
  // deferredQuery "lag behind" query → không block typing
  const results = useMemo(() => filter(data, deferredQuery), [deferredQuery]);
  return <List items={results} />;
}
```

**Khác biệt:** `useTransition` wrap **setter**, `useDeferredValue` wrap **value**. Dùng `useDeferredValue` khi không control được setter (nhận từ props).

---

### Q20. React Server Components (RSC) là gì? Khác gì SSR truyền thống?

**Senior:**

| | SSR | RSC |
|---|---|---|
| **Chạy ở đâu** | Server render → HTML → Client hydrate lại toàn bộ | Component chạy *chỉ* trên server, KHÔNG gửi JS |
| **JS Bundle** | Vẫn gửi toàn bộ JS | Server Components = 0 KB client JS |
| **Interactivity** | Full — sau hydration | Server Components: KHÔNG interactive. Client Components: có |
| **Data fetching** | getServerSideProps / loader | Trực tiếp `await fetch()` trong component |

```javascript
// Server Component (mặc định trong Next.js App Router)
async function ProductPage({ id }) {
  const product = await db.products.findById(id); // truy cập DB trực tiếp
  return <ProductDetails product={product} />;     // render trên server
}

// Client Component — cần 'use client' directive
'use client';
function AddToCartButton({ productId }) {
  const [added, setAdded] = useState(false);
  return <button onClick={() => setAdded(true)}>Add to Cart</button>;
}
```

---

## Phần 6: Patterns & Architecture

---

### Q21. Compound Components pattern — giải thích và cho ví dụ.

**Senior:** Các components chia sẻ implicit state thông qua Context, cho phép user tự do compose UI mà vẫn giữ logic nhất quán:

```javascript
// Usage — clean, declarative API
<Select onChange={handleChange}>
  <Select.Trigger>Choose an option</Select.Trigger>
  <Select.Options>
    <Select.Option value="a">Option A</Select.Option>
    <Select.Option value="b">Option B</Select.Option>
  </Select.Options>
</Select>

// Implementation
const SelectContext = createContext();

function Select({ children, onChange }) {
  const [isOpen, setIsOpen] = useState(false);
  const [selected, setSelected] = useState(null);
  const value = useMemo(
    () => ({ isOpen, setIsOpen, selected, setSelected, onChange }),
    [isOpen, selected, onChange]
  );
  return (
    <SelectContext.Provider value={value}>
      {children}
    </SelectContext.Provider>
  );
}

Select.Trigger = function Trigger({ children }) {
  const { setIsOpen, selected } = useContext(SelectContext);
  return <button onClick={() => setIsOpen(o => !o)}>{selected || children}</button>;
};
// ... Select.Options, Select.Option tương tự
```

**Ưu điểm:** Flexibility cao, user control layout. **Ví dụ thực tế:** Radix UI, Headless UI.

---

### Q22. Render Props vs HOC vs Custom Hooks — evolution ra sao?

**Senior:**

```javascript
// 1. HOC (2015) — wrap component, inject props
const withAuth = (Component) => (props) => {
  const user = useAuth();
  return user ? <Component {...props} user={user} /> : <Login />;
};
// Vấn đề: wrapper hell, prop collision, khó debug

// 2. Render Props (2017) — function as children
<Mouse render={({ x, y }) => <Cursor x={x} y={y} />} />
// Vấn đề: callback hell, awkward JSX nesting

// 3. Custom Hooks (2019+) — ✅ Hiện tại recommended
function useAuth() {
  const [user, setUser] = useState(null);
  useEffect(() => { /* subscribe */ }, []);
  return { user, login, logout };
}
// Sạch, composable, dễ test, không thêm component tree
```

**Kết luận:** Custom Hooks thay thế 90% use cases của HOC và Render Props. HOC vẫn dùng cho cross-cutting concerns đặc biệt (analytics wrapper, error boundary wrapper).

---

### Q23. Error Boundary — tại sao chưa có hooks equivalent?

**Senior:** Error Boundaries bắt lỗi trong **render phase** và **lifecycle methods** — chúng cần `componentDidCatch` và `getDerivedStateFromError` là class methods.

Hooks chạy **sau render**, nên không thể catch lỗi **trong render**. React team chưa giải quyết được vấn đề API design cho hook-based error boundary.

```javascript
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logErrorToService(error, info.componentStack);
  }

  render() {
    if (this.state.hasError) return <FallbackUI />;
    return this.props.children;
  }
}

// Workaround: dùng library react-error-boundary
import { ErrorBoundary } from 'react-error-boundary';
<ErrorBoundary FallbackComponent={ErrorFallback} onError={logError}>
  <App />
</ErrorBoundary>
```

> **Lưu ý:** Error Boundary KHÔNG catch lỗi trong event handlers, async code, SSR. Những lỗi đó cần try/catch thông thường.

---

### Q24. Controlled vs Uncontrolled Components — khi nào dùng uncontrolled?

**Senior:**
- **Controlled:** React quản lý state qua `value` + `onChange`. Mọi keystroke đi qua React.
- **Uncontrolled:** DOM tự quản lý state, React chỉ đọc qua `ref` khi cần.

**Dùng Uncontrolled khi:**
- Form đơn giản, submit một lần (không cần real-time validation)
- File input (`<input type="file">` bắt buộc uncontrolled)
- Integration với non-React library
- Performance: tránh re-render mỗi keystroke (form cực lớn)

```javascript
// Uncontrolled với React Hook Form — best of both worlds
import { useForm } from 'react-hook-form';

function MyForm() {
  const { register, handleSubmit } = useForm();
  return (
    <form onSubmit={handleSubmit(data => console.log(data))}>
      <input {...register('email')} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Phần 7: Testing

---

### Q25. Testing strategy cho React app của bạn như thế nào?

**Senior:** Theo **Testing Trophy** (Kent C. Dodds):

```
        E2E        ← ít test, high confidence, chậm
      /      \
   Integration     ← NHIỀU NHẤT test ở đây — test user flows
  /            \
  Unit Tests       ← test pure logic, utils, reducers
  ──────────────
   Static Types    ← TypeScript catches bugs at compile time
```

- **Unit:** Jest — test reducers, utils, custom hooks (`renderHook`)
- **Integration:** React Testing Library — test component behavior từ góc nhìn user
- **E2E:** Playwright / Cypress — test critical user journeys
- **Static:** TypeScript strict mode

```javascript
// ✅ Test behavior, không test implementation
test('user can submit login form', async () => {
  render(<LoginForm />);

  await userEvent.type(screen.getByLabelText(/email/i), 'test@mail.com');
  await userEvent.type(screen.getByLabelText(/password/i), '123456');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));

  expect(await screen.findByText(/welcome/i)).toBeInTheDocument();
});

// ❌ Anti-pattern: test implementation
expect(component.state.isLoggedIn).toBe(true); // test internal state
expect(wrapper.find('LoginForm').prop('onSubmit')).toHaveBeenCalled(); // test props
```

---

### Q26. Làm sao test custom hooks?

**Senior:**

```javascript
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter(0));

  expect(result.current.count).toBe(0);

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});

// Hook cần Provider → wrapper option
test('hook with context', () => {
  const wrapper = ({ children }) => (
    <AuthProvider>{children}</AuthProvider>
  );
  const { result } = renderHook(() => useAuth(), { wrapper });
  expect(result.current.user).toBeDefined();
});
```

---

### Q27. Mock API calls trong test — approach nào tốt nhất?

**Senior:** **MSW (Mock Service Worker)** — mock ở network level, không cần mock `fetch`/`axios`:

```javascript
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

const server = setupServer(
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' },
    ]);
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test('loads users', async () => {
  render(<UserList />);
  expect(await screen.findByText('John')).toBeInTheDocument();
  expect(screen.getByText('Jane')).toBeInTheDocument();
});

// Test error case
test('shows error on failure', async () => {
  server.use(
    http.get('/api/users', () => {
      return new HttpResponse(null, { status: 500 });
    })
  );
  render(<UserList />);
  expect(await screen.findByText(/error/i)).toBeInTheDocument();
});
```

**Ưu điểm MSW:** Cùng mock dùng cho test, development, Storybook. Không phụ thuộc implementation (fetch vs axios).

---

## Phần 8: TypeScript + React

---

### Q28. Những TypeScript patterns nào quan trọng nhất trong React?

**Senior:**

```typescript
// 1. Discriminated Unions cho props
type ButtonProps =
  | { variant: 'link'; href: string; onClick?: never }
  | { variant: 'button'; onClick: () => void; href?: never };

function Button(props: ButtonProps) {
  if (props.variant === 'link') return <a href={props.href}>...</a>;
  return <button onClick={props.onClick}>...</button>;
}

// 2. Generic Components
type TableProps<T> = {
  data: T[];
  columns: { key: keyof T; header: string }[];
  renderRow: (item: T) => React.ReactNode;
};
function Table<T>({ data, columns, renderRow }: TableProps<T>) { ... }

// 3. ComponentPropsWithRef — extend native element props
type InputProps = React.ComponentPropsWithRef<'input'> & {
  label: string;
  error?: string;
};
const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, ...props }, ref) => (...)
);

// 4. as const cho exhaustive type checking
const ROLES = ['admin', 'user', 'guest'] as const;
type Role = typeof ROLES[number]; // 'admin' | 'user' | 'guest'

// 5. Template Literal Types
type Color = 'primary' | 'secondary';
type Size = 'sm' | 'md' | 'lg';
type ButtonClass = `btn-${Color}-${Size}`;
// "btn-primary-sm" | "btn-primary-md" | ... (6 options)
```

---

### Q29. `React.FC` — nên dùng hay không?

**Senior:** **Không nên dùng.** Kể từ React 18, `React.FC`:
- Không còn tự thêm `children` vào props (React 18 bỏ)
- Không hỗ trợ generics
- Thêm implicit return type không cần thiết

```typescript
// ❌ Không recommended
const MyComponent: React.FC<Props> = ({ title }) => { ... };

// ✅ Recommended — explicit, rõ ràng
function MyComponent({ title }: Props): JSX.Element { ... }
// hoặc
const MyComponent = ({ title }: Props) => { ... };
```

---

## Phần 9: System Design & Tình Huống

---

### Q30. Thiết kế một real-time dashboard hiển thị 50 charts, mỗi chart update mỗi giây. Làm sao không lag?

**Senior:** Approach multi-layer:

1. **Data layer:**
   - WebSocket connection duy nhất, server push chỉ data thay đổi (delta updates)
   - Normalize data, chỉ update chart nào có data mới

2. **Rendering layer:**
   - Mỗi chart là một `React.memo` component với custom comparator
   - Dùng `useDeferredValue` cho non-visible charts
   - Canvas-based chart library (không DOM nodes) — lightweight hơn SVG charts

3. **Layout:**
   - Virtualization — chỉ render charts trong viewport
   - `requestAnimationFrame` batching — accumulate updates, render 1 lần/frame

4. **Architecture:**
   ```
   WebSocket → SharedWorker (parse/compute) → Zustand Store → React.memo Charts
   ```
   - SharedWorker tách computation khỏi main thread
   - Zustand với selectors → mỗi chart subscribe đúng slice data của mình

---

### Q31. User report rằng search box bị lag khi gõ. Cách debug và fix?

**Senior:** Step by step:

1. **Profile:** React DevTools Profiler → xem component nào re-render lâu nhất khi gõ
2. **Likely causes:**
   - Filter/search chạy trên large dataset mỗi keystroke
   - Search results re-render toàn bộ list
   - API call mỗi keystroke

3. **Fix theo mức độ:**
```javascript
// Fix 1: Debounce API call
const debouncedQuery = useDebounce(query, 300);
useEffect(() => { fetchResults(debouncedQuery); }, [debouncedQuery]);

// Fix 2: useTransition cho client-side filter
const [isPending, startTransition] = useTransition();
function handleSearch(e) {
  setQuery(e.target.value);       // urgent
  startTransition(() => {
    setResults(filterData(e.target.value)); // non-urgent
  });
}

// Fix 3: Virtualize results list
<VirtualList items={results} itemHeight={40} />

// Fix 4: Web Worker cho heavy computation
const worker = new Worker('./searchWorker.js');
worker.postMessage({ query, data });
worker.onmessage = (e) => setResults(e.data);
```

---

### Q32. Làm sao bạn structure một React project lớn (20+ developers)?

**Senior:**

```
src/
├── app/                    # App-level setup (routes, providers, layouts)
│   ├── routes.tsx
│   ├── providers.tsx
│   └── layouts/
│
├── features/               # Feature-based modules (domain logic)
│   ├── auth/
│   │   ├── components/     # Feature-specific components
│   │   ├── hooks/          # Feature-specific hooks
│   │   ├── api/            # API calls
│   │   ├── store/          # Feature state (Zustand slice / Redux slice)
│   │   ├── types.ts
│   │   └── index.ts        # Public API — chỉ export cái cần thiết
│   ├── dashboard/
│   └── orders/
│
├── shared/                 # Shared across features
│   ├── components/         # Button, Modal, Input, ...
│   ├── hooks/              # useDebounce, useLocalStorage, ...
│   ├── utils/              # Pure utility functions
│   ├── api/                # API client, interceptors
│   └── types/              # Global types
│
└── config/                 # Env, constants, feature flags
```

**Nguyên tắc:**
- Feature modules **không import lẫn nhau** — chỉ import từ `shared/`
- Mỗi feature có `index.ts` làm **public API** — encapsulation
- Monorepo nếu cần: Nx/Turborepo
- Barrel exports ở feature level, KHÔNG ở `shared/` (tree-shaking issues)

---

## Phần 10: Gotcha & Edge Cases

---

### Q33. `key` prop ngoài list rendering thì còn dùng để làm gì?

**Senior:** **Reset component state!** Thay đổi `key` = React unmount component cũ, mount component mới hoàn toàn:

```javascript
// Reset form khi chuyển user
<UserProfile key={userId} user={user} />

// Force re-create animation
<AnimatedComponent key={animationId} />

// Reset uncontrolled input
<input key={formVersion} defaultValue="initial" />
```

Đây là cách **đơn giản nhất** để reset internal state mà không cần `useEffect` cleanup phức tạp.

---

### Q34. Memory leaks phổ biến nhất trong React app?

**Senior:**

```javascript
// 1. ❌ Event listener không cleanup
useEffect(() => {
  window.addEventListener('resize', handler);
  // Thiếu: return () => window.removeEventListener('resize', handler);
}, []);

// 2. ❌ Subscription không unsubscribe
useEffect(() => {
  const sub = eventBus.subscribe('event', handler);
  // Thiếu: return () => sub.unsubscribe();
}, []);

// 3. ❌ setState sau unmount
useEffect(() => {
  fetchData().then(data => {
    setData(data); // component có thể đã unmount
  });
  // Fix: AbortController
  const controller = new AbortController();
  fetchData({ signal: controller.signal }).then(setData).catch(() => {});
  return () => controller.abort();
}, []);

// 4. ❌ setInterval không clear
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);  // PHẢI có
}, []);

// 5. ❌ Closure giữ reference lớn
// Large object bị capture trong closure → GC không thu hồi được
```

---

### Q35. `dangerouslySetInnerHTML` — khi nào dùng và cách dùng an toàn?

**Senior:** Dùng khi cần render raw HTML (CMS content, markdown rendered, rich text editor output).

```javascript
import DOMPurify from 'dompurify';

// ✅ LUÔN sanitize trước khi render
function RichContent({ htmlContent }) {
  const sanitized = DOMPurify.sanitize(htmlContent, {
    ALLOWED_TAGS: ['p', 'b', 'i', 'a', 'ul', 'li', 'br', 'h1', 'h2', 'h3'],
    ALLOWED_ATTR: ['href', 'target'],
  });
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}

// ❌ NEVER: render unsanitized user input
<div dangerouslySetInnerHTML={{ __html: userInput }} /> // XSS attack vector
```

---

### Q36. Tại sao `{...props}` spreading có thể nguy hiểm?

**Senior:**
```javascript
// ❌ Truyền unknown props xuống DOM → React warning
function Card(props) {
  return <div {...props} />; // nếu props có isActive, onCustomEvent → DOM warning
}

// ✅ Destructure known props, rest cho DOM
function Card({ isActive, onCustomEvent, className, ...domProps }) {
  return (
    <div
      className={`card ${isActive ? 'active' : ''} ${className}`}
      {...domProps}
    />
  );
}
```

---

### Q37. Giải thích sự khác biệt giữa `createElement` và JSX.

**Senior:** JSX chỉ là **syntactic sugar** — Babel/SWC compile JSX thành `createElement` calls:

```javascript
// JSX
<Button color="primary" onClick={handleClick}>
  Submit
</Button>

// Compiled (React 16)
React.createElement(Button, { color: 'primary', onClick: handleClick }, 'Submit');

// Compiled (React 17+ với new JSX transform)
import { jsx as _jsx } from 'react/jsx-runtime';
_jsx(Button, { color: 'primary', onClick: handleClick, children: 'Submit' });
```

React 17+ dùng **new JSX transform** → không cần `import React` ở mỗi file nữa.

---

### Q38. `Suspense` hoạt động như thế nào internally?

**Senior:** Suspense dựa trên cơ chế **throw Promise**:

1. Component con **throw một Promise** khi data chưa sẵn sàng
2. Suspense boundary **catch** Promise đó → render fallback
3. Khi Promise resolve → React **retry render** component con
4. Component con render thành công → Suspense hiển thị content

```javascript
// Đây là cách React.lazy hoạt động internally
function lazy(importFn) {
  let status = 'pending';
  let result;
  const promise = importFn().then(
    module => { status = 'success'; result = module; },
    error => { status = 'error'; result = error; }
  );

  return function LazyComponent(props) {
    if (status === 'pending') throw promise;   // Suspense catches this
    if (status === 'error') throw result;       // Error Boundary catches this
    return <result.default {...props} />;
  };
}
```

---

### Q39. Hydration mismatch là gì? Làm sao tránh?

**Senior:** Khi HTML được server render **khác** với HTML mà client render ra → React warning/error khi hydrate.

```javascript
// ❌ Gây mismatch
function Component() {
  return <p>{Date.now()}</p>;           // khác value server vs client
  return <p>{Math.random()}</p>;         // khác value server vs client
  return <p>{window.innerWidth}</p>;     // window không tồn tại trên server
}

// ✅ Fix
function Component() {
  const [mounted, setMounted] = useState(false);
  useEffect(() => setMounted(true), []);

  if (!mounted) return <p>Loading...</p>; // match server HTML
  return <p>{window.innerWidth}px</p>;    // chỉ render trên client
}

// ✅ React 18: suppressHydrationWarning cho intentional mismatch
<time suppressHydrationWarning>{new Date().toLocaleString()}</time>
```

---

### Q40. Bạn sẽ migrate một React class component codebase sang hooks như thế nào?

**Senior:** **Incremental migration**, KHÔNG rewrite toàn bộ:

1. **Không đụng code đang chạy tốt** — class components vẫn được support
2. **New features viết bằng hooks** — tất cả code mới dùng functional components
3. **Migrate khi có reason** — refactor, bug fix, thêm feature → chuyển sang hooks luôn
4. **Extract logic thành custom hooks trước** — tách business logic ra, component chỉ còn render
5. **Test coverage trước khi migrate** — đảm bảo behavior không đổi

```
Priority order:
1. Shared logic → Custom Hooks (reusable nhất)
2. Simple components (ít state) → Quick wins
3. Complex components → Cẩn thận, test kỹ
4. Components dùng componentDidCatch → GIỮ class (chưa có hook equivalent)
```

> **Tuyệt đối không** tạo PR "migrate tất cả 200 components sang hooks". Đó là recipe for disaster.

---

## Bonus: Quick-Fire Round 🔥

| Câu hỏi | Trả lời ngắn |
|---|---|
| `useId()` dùng để làm gì? | Generate stable unique ID cho accessibility (htmlFor/id), work với SSR |
| React.Fragment vs `<></>` | `<>` không nhận props. `<React.Fragment key={id}>` khi cần key |
| StrictMode làm gì? | Double-invoke effects + render (dev only) để detect side effects |
| Portals dùng khi nào? | Render DOM node ngoài parent (modals, tooltips, dropdowns) |
| `flushSync` dùng khi nào? | Force synchronous re-render (hiếm dùng — DOM measurement sau state update) |
| Tại sao immutability quan trọng? | React dùng reference equality check — mutate object = React không biết thay đổi |
| Synthetic Events là gì? | React wrap native events → normalize cross-browser. React 17+ gắn vào root, không document |
| Prop drilling giải quyết bằng gì? | Context API, Zustand, Component Composition (truyền component thay vì data) |
