Ran into the following issue when working on changing the node of an AST:

```rust
use std::borrow::BorrowMut;

struct A {
    child: Option<Box<B>>
}

enum B {
    A(A),
    B,
}

fn change(b: &mut B) {
    match b {
        B::A(a) => {
            // What's the correct way to abstract this to a function?

            // works
            match &mut a.child {
                Some(b_box) => {
                    change(b_box.borrow_mut());
                },
                None => {
                    *b = B::B;
                }
            };

            // results in "cannot borrow `*b` as mutable more than once at a time"
            change_inner(b, a);
        }
        B::B => (),
    };
}

fn changeuse std::borrow::BorrowMut;
use std::mem::replace;

struct A {
    child: Option<Box<B>>,
}

enum B {
    A(A),
    B,
}

fn change(b: &mut B) {
    let mut temp_b = replace(b, B::B);
    match &mut temp_b {
        B::A(a) => {
            change_inner(b, a);
            *b = temp_b;
        }
        B::B => (),
    };
}

fn change_inner(b: &mut B, a: &mut A) {
    match &mut a.child {
        Some(b_box) => {
            change(b_box.borrow_mut());
        }
        None => {
            *b = B::B;
        }
    };
}

fn main() {
    let mut s1 = B::A(A { child: None });

    change(&mut s1);
}_inner(b: &mut B, a: &mut A) {
    match &mut a.child {
        Some(b_box) => {
            change(b_box.borrow_mut());
        },
        None => {
            *b = B::B;
        }
    };
}

fn main() {
    let mut s1 = B::A(A { child: None });

    change(&mut s1);
}
```

In short, I was trying to do the following:
1. Pass a mutable reference of a node as well as one of its children to a function.
1. Based on the value of the child node, mutate the parent node.

After asking in the Rust Discord, scottmcm told me about std::mem::replace. Basically the idea is to swap out the node in question so that we have ownership of the node, and then we can do as we please with it. At the end of our operation, assign it back.

After:
```rust
use std::borrow::BorrowMut;
use std::mem::replace;

struct A {
    child: Option<Box<B>>,
}

enum B {
    A(A),
    B,
}

fn change(b: &mut B) {
    let mut temp_b = replace(b, B::B);
    match &mut temp_b {
        B::A(a) => {
            change_inner(b, a);
            *b = temp_b;
        }
        B::B => (),
    };
}

fn change_inner(b: &mut B, a: &mut A) {
    match &mut a.child {
        Some(b_box) => {
            change(b_box.borrow_mut());
        }
        None => {
            *b = B::B;
        }
    };
}

fn main() {
    let mut s1 = B::A(A { child: None });

    change(&mut s1);
}
```
