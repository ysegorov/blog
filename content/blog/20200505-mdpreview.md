+++
path = "/2020/mdpreview/"
title = "Markdown preview (Rust version)"
date = 2020-05-05
[extra]
modified = 2020-05-05T07:38:00+00:00
+++
# Markdown preview written in Rust

About a year or so ago I've switched to self-made python script to preview
markdown file in browser ([this commit][mdpreview-python] introduced it in my
own [dotfiles][dotfiles] repository).

It was using [pandoc][pandoc] under the hood to convert markdown to html. Well,
it was not optimal, probably, but it worked.

I've started to read [Rust][rust] documentation recently and decided to
(re)implement markdown preview in Rust.

The most valuable thing I saw in this was the fact that it will be single
binary to use - no third-party dependencies would be needed to install.

So, here is [the code][mdpreview-rust] - markdown preview written in Rust.

This is my first project written in Rust and it is definitely not as idiomatic
as it should be and has no tests at the moment but it works as expected. Yay.

Under the hood [mdpreview][mdpreview-rust] uses following Rust libraries:

- [clap][clap] - to process command-line arguments,
- [pulldown-cmark][pulldown-cmark] - to convert markdown to html,
- [hyper][hyper] and [tokio][tokio] - web server to serve generated html.

Some ideas for the code I've got looking through sources of [zola][zola].

[Ownership][rust-ownership] is a new concept to me and I'm still not sure I'm
getting it right but Rust is definitely worth trying. Here is the list of
resources I'm using to study Rust (just for the record):

- [The Book][rust-the-book],
- [Rust By Example][rust-by-example],
- [Rust Cookbook][rust-cookbook],
- [The Standard Library][rust-std],
- [crates.io][crates.io],
- [docs.rs][docs.rs].

That's it for the [mdpreview][mdpreview-rust] intro, let's move on and
(re)implement in Rust something else. Stay tuned.

[mdpreview-python]: https://github.com/ysegorov/dotfiles/commit/3e54334f3afbb2f79b1536661ac5c7e469228d51#diff-222c971ff832264e6f73fa6475e6ffeb
[dotfiles]: https://github.com/ysegorov/dotfiles
[pandoc]: https://pandoc.org
[rust]: https://www.rust-lang.org
[mdpreview-rust]: https://github.com/ysegorov/mdpreview-rs
[clap]: https://docs.rs/clap/
[pulldown-cmark]: https://docs.rs/pulldown-cmark/
[hyper]: https://docs.rs/hyper/
[tokio]: https://docs.rs/tokio/
[zola]: https://www.getzola.org
[rust-ownership]: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
[rust-the-book]: https://doc.rust-lang.org/book/
[rust-by-example]: https://doc.rust-lang.org/stable/rust-by-example/
[rust-cookbook]: https://rust-lang-nursery.github.io/rust-cookbook/intro.html
[rust-std]: https://doc.rust-lang.org/std/index.html
[crates.io]: https://crates.io
[docs.rs]: https://docs.rs
