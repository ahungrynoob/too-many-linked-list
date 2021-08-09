# 笔记

> https://rust-unofficial.github.io/too-many-lists/first-layout.html

## A Persistent Stack

### Option.and_then

效果相当于 flatmap, 如果 f 返回的是 Option，最终就只会有一层 Option, 不会成为 `Option<Option<T>>`
