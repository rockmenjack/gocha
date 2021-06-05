# gotcha 避坑手册
## golang
### unreleased resource with blank identifier 你忽略的变量可能会引起资源泄漏
Sometimes you might ignore some of the return values from a function call, because you don't need them:
```go
val, err := f()
// if val is never used, then we might change to:
_, err := f()
```
however if for example `val.Close()` must be called to release the underlying resources, then you are leaking with blank identifier

## Rust

these rules/knowledge may be out of date

### be careful about {} block returning a value

```rust
let inner = &mut unsafe { (*self.inner).replace }
```
is effectively
```rust
let tmp = unsafe { (*self.inner).replace };
let inner = &mut tmp //reference to stack data
```
In `unsafe` context, pointer deferencing will subvert borrow checker, so you might pass a value in stack to another function. then boom, your program panic!

refer: https://www.reddit.com/r/rust/comments/nqjyb7/the_most_annoying_bug_ive_had_to_track_down/

### auto-referencing rules
1. Is there an impl for T , if so use that impl, else go to step 2
2. Is there an impl for &T , if so use that impl, else go to step 3,
3. Is there an impl for &mut T , if so use that impl, else go to step 4
4. Is T: Deref, then try dereferencing (this may fail if <T as Deref>::Target is not Copy or of T is not Copy)
5. there may be others
refer:
https://internals.rust-lang.org/t/restrict-auto-ref-for-method-taking-self/10909/11?u=rockmen1

### the correct way of working with sub-string of string in unicode
use https://crates.io/crates/unicode-segmentation

refer:
https://users.rust-lang.org/t/how-to-get-a-substring-of-a-string/1351

### rust module system explained in one sentence
We need to explicitly build the module tree in Rust - there’s no implicit mapping between file system tree to module tree.

refer:
http://www.sheshbabu.com/posts/rust-module-system/


### return always exit a fn
```rust
fn test() -> Option<u8> {
    let v = {
        let n : u8 = rand::thread_rng().gen();
        if n > 10 {
            n
        } else if n == 0 {
            return None; // function exits
        } else {
            1
        }
    };

    Some(v)
}
```

### drop orders are different among scenarios
+ for variable binding and `if`, drop is called right after it is out of scope
```rust
impl Foo {
    fn do_something(&mut self) {
        let _ = self.lock.lock();    // <- lock will drop immediately, which is very bad!
        self.resource.use_somehow();
    }
}

...
    if make_token("if token").is_ok() { // <- variable created by make_token will drop immediately right before {} block
        println!("if body");
    }
...
```
+ for other cases drop will be called after extened scope

refer:
https://vojtechkral.github.io/blag/rust-drop-order/
