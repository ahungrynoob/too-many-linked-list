# 笔记

> https://rust-unofficial.github.io/too-many-lists/first-layout.html

## A Bad Stack

### Layout

考虑这种数据结构:

```rust
pub enum List {
    Empty,
    Elem(i32, Box<List>),
}
```

在内存中的存储方式：

```
[] = Stack
() = Heap

[Elem A, ptr] -> (Elem B, ptr) -> (Empty, *junk*)
```

缺点：

1. 第一个元素存储在了栈上
2. 最后一个元素，浪费了一个指针位置去指向 emtpy (虽然 rust 有空指针优化，但还是不够优雅）
3. 不纯的分布对操作元素来说不够统一

```
layout 1:

[Elem A, ptr] -> (Elem B, ptr) -> (Elem C, ptr) -> (Empty *junk*)

split off C:

[Elem A, ptr] -> (Elem B, ptr) -> (Empty *junk*)
[Elem C, ptr] -> (Empty *junk*)
```

比较好的内存存储方式

```
[ptr] -> (Elem A, ptr) -> (Elem B, *null*)
```

下面的这种数据结构实现了上面的那种内存布局

> We need to better separate out the idea of having an element from allocating another list. To do this, we have to think a little more C-like: structs!

```rust
pub struct List {
    head: Link,
}

enum Link {
    Empty,
    More(Box<Node>),
}

struct Node {
    elem: i32,
    next: Link,
}

```

### Drop

rust 语言默认的 drop 行为在单链表的数据结构中，会递归 drop， 但是遇到 `Box<Node>` 结构的时候，不会有尾递归优化，因为

```rust
impl Drop for Box<Node> {
    fn drop(&mut self) {
        self.ptr.drop(); // uh oh, not tail recursive!
        deallocate(self.ptr);
    }
}
```

所以为了防止 stack 爆掉，我们需要自己实现 drop

```rust
impl Drop for List {
    fn drop(&mut self) {
        let mut cur_link = mem::replace(&mut self.head, Link::Empty);
        // `while let` == "do this thing until this pattern doesn't match"
        while let Link::More(mut boxed_node) = cur_link {
            cur_link = mem::replace(&mut boxed_node.next, Link::Empty);
            // boxed_node goes out of scope and gets dropped here;
            // but its Node's `next` field has been set to Link::Empty
            // so no unbounded recursion occurs.
        }
    }
}
```
