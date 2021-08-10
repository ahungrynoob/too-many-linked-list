## A Bad Safe Dequeue

### RefCell

1. 被 RefCell 包裹的类型，没办法直接访问属性
2. into_inner - Consumes the RefCell, returning the wrapped value.
3. borrow 和 borrow_mut 返回的 Ref 和 RefMut 的声明周期必须要比 & 和 &mut 长

```rust
fn borrow<'a>(&'a self) -> Ref<'a, T>
fn borrow_mut<'a>(&'a self) -> RefMut<'a, T>
```

> Ref and RefMut implement Deref and DerefMut respectively. So for most intents and purposes they behave exactly like &T and &mut T. However, because of how those traits work, the reference that's returned is connected to the lifetime of the Ref, and not the actual RefCell. This means that the Ref has to be sitting around as long as we keep the reference around.

### Ref

1. Ref 也可以 map, 把一个 Ref<Node<T>> 映射成 Ref<T>

```rust
pub fn peek_front(&self) -> Option<Ref<T>> {
    self.head.as_ref().map(|node| {
        Ref::map(node.borrow(), |node| &node.elem)
    })
}
```

2. Ref::map_slipt - Splits a Ref into multiple Refs for different components of the borrowed data.

### Cell

实现了 copy trait 的需要用 Cell 去运行时改变，其他类型需要用 RefCell

### When To Use RefCell

1. Introducing inherited mutability roots to shared types.
2. Implementation details of logically-immutable methods.
3. Mutating implementations of Clone.

> Shared smart pointer types, including Rc<T> and Arc<T>, provide containers that can be cloned and shared between multiple parties. Because the contained values may be multiply-aliased, they can only be borrowed as shared references, not mutable references. Without cells it would be impossible to mutate data inside of shared boxes at all! It's very common then to put a RefCell<T> inside shared pointer types to reintroduce mutability:

```rust
use std::collections::HashMap;
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let shared_map: Rc<RefCell<_>> = Rc::new(RefCell::new(HashMap::new()));
    shared_map.borrow_mut().insert("africa", 92388);
    shared_map.borrow_mut().insert("kyoto", 11837);
    shared_map.borrow_mut().insert("piccadilly", 11826);
    shared_map.borrow_mut().insert("marbles", 38);
}
```

### Rc 和 Arc

Rc 是在单线程中使用的， Arc 是在多线程中使用, `atomic reference count`

### Mutex<T> 和 RefCell<T>

RefCell<T>s are for single-threaded scenarios. Consider using Mutex<T> if you need shared mutability in a multi-threaded situation.
