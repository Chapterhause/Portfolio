A **package** is a Rust project with a `Cargo.toml` file, `Cargo.lock` file, and some number of crates. 

A **crate** can be either a binary crate, which has a `main.rs` file (which has a `fn main()` function), and produces a binary executable, or a library crate, which has a `lib.rs` file, and produces no binary when it compiles, but instead functions as a library. A binary crate can also contain a `lib.rs` file, purely for the benefit of its `main.rs` file. 

**Modules** are organizational groups of code in a crate that could include function, [[Structs|struct]], and [[Enums|enum]] definitions, which can then be accessed by the **crate root**, which is that initial `main.rs` or `lib.rs` file. Modules can be nested within curly braces in the file in the crate root, or receive their own file, within certain limitations. They provide ways to organize a very code-heavy crate, by keeping it readable and localized in different files. Also, the functions and types defined in modules can be used in the definitions of other functions and types in the module's parents. 

When using the command `cargo new project_name`, a new package called `project_name` is created, and then a library crate in `/src`, with the same name as the package. Adding the flag `--bin` will instead create a binary create in `/src`. 

From here, modules can be defined in the crate root's file with the `mod` keyword. Then, the module could be defined inside brackets immediately after that. 
```
// in main.rs

mod Math {

	fn Sum (x: u32, y: u32) -> u32 {
		x+y
	}

}

fn main() {
	// something going on in here
}
```
*The module `Math` could hold another module, defined in the same way, in addition to the function `Sum`*

Modules always have to be declared with `mod` in a parent file, but they do not have to be defined in that file. The compiler will also look for a definition in a file with the same name as the module. 
```
/project_name
	Cargo.toml
	Cargo.lock 
	/src
		main.rs
		math.rs
```
*The module `Math` was declared in `main.rs` as `mod Math;` and then defined in `math.rs`*

If a module contains **submodules**, and it is defined within curly braces, those submodules could then be defined within their own set of braces inside that. 
```
//main.rs 

mod Math {
	mod Squaring {

		fn Square (x: u32) -> u32 {
			x*x
		}

	}

	fn Sum (x: u32, y: u32) -> u32 {
		x+y
	}
}
```

If that parent module were instead declared in the crate root and then defined in its own file, the submodule could be defined in curly braces within *that* file, or declared in its own file, named after itself, in a directory named after the parent module. 
```
project_name
	Cargo.toml
	Cargo.lock
	/src
		main.rs
		math.rs
		/math
			squaring.rs
```


The functions, structs, and enums defined within modules can be accessed by parent modules and the crate root by using the **path** to that object. For `main.rs` to access `fn Square()` function in the above module tree, the path to that function would have to be used. 
```
// in main.rs

fn main() {
	let x = 5; 
	let y = crate::Math::Squaring::Square(x);
}
```
*`Crate` means that the path starts from the root of the crate. Then the module `Math`, which is the parent of `Square()`'s parent module, must be included. Then the function's parent module, `Squaring`, then the function. All separated by `::`*

Note that even though `crate` can be used to represent the root of the crate, it is really a substitute for the crate name. If another crate was trying to access `Square()`, which it could, it would need to use the path `math_crate::Math::Squaring::Square()`, where `math_crate` is the name of the crate with `Math`. 

The above is an **absolute** path, because it starts from the root of the crate, then goes through both parent modules, then the function in question. It is also possible to use a **relative** path, which starts from the current position in the module tree, instead of the root of the crate. 
```
// in math.rs

mod Squaring; 

fn Sum (x: u32, y: u32) -> u32 {
	x+y
}

fn AreaOfSquare (sidelength: u32) -> u32 {
	Squaring::Square(sidelength)
}
```
*In this case, the module that the function comes from is a child of `AreaOfSquare`'s parent, so it can be accessed just by specifying that sibling of `AreaOfSquare`, and than that sibling's child function*

Using the above example, if `Squaring` was the child of the module `Utilities` instead of `Math`, meaning `mod Utilities;` was declared in `main.rs`, in addition to `mod Math;`, the relative path would be `Utilities::Squaring::Square(sidelength)`. 

Now, all the previous examples of using paths to access the elements of modules would not compile, because of Rust's **privacy** rules. Essentially, by default, the children of modules cannot be accessed by their ancestors. So `main.rs` cannot by default use the `Square()` function, and neither can `AreaOfSquare`. However, modules can access what their ancestors have, so `Square()` could use the function `Sum()` in its definition. To explicitly allow ancestors to borrow from their children, modules and their children can be made **public** with the `pub` keyword. If `Math` is defined as `pub mod Math;`, it is made public. However, that does not necessarily mean its *contents* are public. Only if the `Sum()` function is similarly defined with the `pub` keyword can `main.rs` use `crate::Math::Sum()`. If, in addition to `Math`, the `Squaring` module and the `Square()` function are public, then `main.rs` will be able to access `crate::Math::Squaring::Square()`. 

Note: even if structs are made public, their fields are not necessarily public. Each field can be made public, or otherwise stay private. The same does *not* apply for the variants of enums, for obvious reasons.

Finally, the paths to access module children can be shortened with the `use` keyword. Assuming everything in the `Math` module is public, and `Math` is public, the following could be included in `main.rs` before `fn main()`. 
```
use crate::Math::Squaring;
```
*From then on in `main.rs`, the `Square()` function could be referred to as `Squaring::Square()` instead of `crate::Math::Squaring::Square()`.*

The `use` keyword can be used not only in the crate root but in the file of any module. 

When not used carefully, the `use` keyword can create naming conflicts. If there is a function defined in `main.rs` also called `Sum()`, then the function in `Math` should just be called `crate::Math::Sum()`. To shorten path names while still avoiding naming conflicts, the `as` keyword can be used to rename an accessed object. 
```
use crate::Math::Sum() as Addition(); 
```
*Calling `Addition()`, if it is not defined already in `main.rs`, will then be like calling `crate::Math::Sum()`.*

Any short-hands created by `pub use` will be made public, so they could be used by ancestors to the current file. However, if `pub use Squaring::Square();` was added to `math.rs`, then `Sum()` could refer to `Square()` as merely `Square()`, but in `main.rs`, `Square()` would be `crate::Math::Square()`, because `Square()` still has `Math` attached to its path. 

To use **external packages** in a package, add to `Cargo.toml` the name of the package, and then its version as a string literal (without a `;`). Then add to the crate root `use package_name::child` to use a child of the package. 
```
// In Cargo.toml
package_name = "0.9.7"
```

The `use` keyword supports nested paths. 
```
//main.rs 

use crate::Math::{Squaring::Square(), Sum()};
```
*Now Square() and Sum() can be referred to simply by their names, because both were `use`-d at the same time*

The `*` or glob operator can also be combined with `use`. It means that the paths of all the children of a certain path should be shortened. 
```
use crate::Math::*;
```
*`Sum()` is just `Sum()`, and `Square()` is just `Squaring::Square()`*. 