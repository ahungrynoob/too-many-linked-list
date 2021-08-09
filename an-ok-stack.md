## Peek

### Option.map

`Option.map` 会拿出 Option 中的 value, 需要返回引用类型的话，就需要用 `as_ref` api

### 三种省略生命周期的情况

```rust
// Only one reference in input, so the output must be derived from that input
fn foo(&A) -> &B; // sugar for:
fn foo<'a>(&'a A) -> &'a B;

// Many inputs, assume they're all independent
fn foo(&A, &B, &C); // sugar for:
fn foo<'a, 'b, 'c>(&'a A, &'b B, &'c C);

// Methods, assume all output lifetimes are derived from `self`
fn foo(&self, &B, &C) -> &D; // sugar for:
fn foo<'a, 'b, 'c>(&'a self, &'b B, &'c C) -> &'a D;
```

### Option<&mut T>.map

和 & 类型不同， &mut 是没有实现 copy 的，因为可以改变的内存的内存地址如果可以 copy，会导致冲突
